# TypeScript Guidelines

These guidelines specifically apply to TypeScript.

## Types

TypeScript provides a range of syntax with which to communicate type information with the compiler.

- The compiler performs **type inference** on all types and values in the code.
- The user can assign **type annotations** (`:`) to override some of the inferred types.
- The user can also add **type assertions** (`as`, `!`) to force the compiler to adhere to the user-supplied type even if it contradicts the compiler's inferences.
- Finally, there are **compiler directives** for disabling type checking (`any`, `@ts-expect-error`) for a limited scope of code.

The general order of preference for using these features coincides with the order in which they were just presented.

### Type Inference

TypeScript is very good at inferring types. Explicit type annotations and assertions are the exception rather than the norm in a well-managed TypeScript codebase.

Some fundamental type information must always be supplied by the user, such as function and class signatures, interfaces for interacting with external entities or data types, and types that express the domain model of the codebase.

However, for the remaining majority of types and values, inference should be preferred over annotations and assertions.

##### Prefer type inference over annotations and assertions

- Explicit type annotations (`:`) and type assertions (`as`, `!`) prevent further inference-based narrowing of the user-supplied types.
  - The compiler errs on the side of trusting user input, which prevents it from utilizing additional type information that it is able to infer.
- Type inferences are responsive to changes in code without requiring user input, while annotations and assertions rely on hard-coding, making them brittle against code drift.
- The `as const` operator can be used to narrow an inferred abstract type into a specific literal type. It can also be used on arrays and objects to achieve the same effect on their properties.

##### Avoid unintentionally widening an inferred type with a type annotation

Enforcing a wider type defeats the purpose of adding an explicit type declaration, as it _loses_ type information instead of adding it. Double-check that the declared type is narrower than the inferred type.

🚫 Type declarations

```typescript
const name: string = 'METAMASK'; // Type 'string'

const chainId: string = this.messagingSystem(
  'NetworkController:getProviderConfig',
).chainId; // Type 'string'

const BUILT_IN_NETWORKS = new Map<string, `0x${string}`>([
  ['mainnet', '0x1'],
  ['sepolia', '0xaa36a7'],
]); // Type 'Map<string, `0x${string}`>'
```

✅ Type inferences

```typescript
const name = 'METAMASK'; // Type 'METAMASK'

const chainId = this.messagingSystem(
  'NetworkController:getProviderConfig',
).chainId; // Type '`0x${string}`'

const BUILT_IN_NETWORKS = {
  mainnet: '0x1',
  sepolia: '0xaa36a7',
} as const; // Type { readonly mainnet: '0x1'; readonly sepolia: '0xaa36a7'; }
```

```typescript
type TransactionMeta = TransactionBase &
  (
    | {
        status: Exclude<TransactionStatus, TransactionStatus.failed>;
      }
    | {
        status: TransactionStatus.failed;
        error: TransactionError;
      }
  );

const updatedTransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected,
};

this.messagingSystem.publish(
  `${controllerName}:transactionFinished`,
  updatedTransactionMeta, // Expected type: 'TransactionMeta'
);
// Property 'error' is missing in type 'typeof updatedTransactionMeta' but required in type '{ status: TransactionStatus.failed; error: TransactionError; }'.ts(2345)
```

🚫 Add type annotation

```typescript
// Type 'TransactionMeta'
const updatedTransactionMeta: TransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected,
}; // resolves error
```

✅ Add `as const` and leave to inference

```typescript
// Type narrower than 'TransactionMeta': { status: TransactionStatus.rejected; ... }
// Doesn't include 'error' property. Correctly narrowed between `TransactionMeta` discriminated union
const updatedTransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected as const,
}; // resolves error
```

### Type Annotations

An explicit type annotation is acceptable for overriding an inferred type if...

1. It can further narrow an inferred type, thus supplying type information that the compiler cannot infer or access.
2. It is determined to be the most accurate _and_ specific type assignable.

##### Use the `satisfies` operator to enforce a type constraint or to validate the assigned type

It is possible to use type annotations in two different ways: to assign a type definition, or to enforce a type constraint.

In the second case, an abstract, "narrowest supertype" is used to constrain or validate a type. Since v4.9, TypeScript provides the `satisfies` operator for this use case. It is able to both enforce a type constraint and further narrow the assigned type through inference.

🚫 Use a type annotation for type validation

```typescript
const updatedTransactionMeta: TransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected,
};

updatedTransactionMeta.error; // Type 'TransactionError'
```

✅ Use the `satisfies` operator for type validation

```typescript
const updatedTransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected as const,
} satisfies TransactionMeta;

updatedTransactionMeta.error; // Type 'undefined'
```

#### Acceptable usages of `:` annotations

##### To prevent or fix type assertions

Type annotations are more responsive to code drift than assertions. If the assignee's type becomes incompatible with the assigned type annotation, the compiler will raise a type error, whereas in most cases a type assertion will still suppress the error.

When the compiler is in doubt, an annotation will nudge it towards relying on type inference, while an assertion will force it to accept the user-supplied type.

<!-- TODO: Add example -->

##### When instantiating an empty container type, provide a type annotation

This is one case where type inference is unable to reach a useful conclusion without user-provided information. Since the compiler cannot arbitrarily restrict the range of types that could be inserted into the container, it has to assume the widest type, which is often `any`. It's up to the user to narrow that into the intended type by adding an explicit annotation.

🚫

```typescript
const tokens = []; // Type 'any[]'
const tokensMap = new Map(); // Type 'Map<any, any>'
```

✅

```typescript
const tokens: string[] = []; // Type 'string[]'
const tokensMap = new Map<string, Token>(); // Type 'Map<string, Token>'
```

### Type Assertions

`as` assertions are unsafe. They overwrite type-checked and inferred types with user-supplied types that suppress compiler errors.

They should only be introduced into the code if the accurate type is unreachable through other means.

##### Document safe or necessary use of type assertions

When a type assertion is absolutely necessary due to constraints or is even safe due to runtime checks, we should document the reasoning behind its usage in the form of a comment.

<!-- TODO: Add example -->

#### Avoid `as`

Type assertions make the code brittle against changes.

While TypeScript and ESLint will flag some unsafe, structurally unsound, or redundant type assertions, they will generally accept user-supplied types without further type-checking.

This can cause silent failures or false negatives where errors are suppressed. This is especially damaging as the codebase accumulates changes over time. Type assertions may continue to silence errors, even though the type itself, or the type's relationship to the rest of the code may have been altered so that the asserted type is no longer valid.

##### Redundant or unnecessary `as` assertions are not flagged for removal

Type assertions can also cause false positives, because they assertions are independent expressions, untied to the type errors they were intended to fix. Thus, even if code drift fixes or removes a particular type error, the type assertions that were put in place to fix that error will provide no indication that they are no longer necessary and now should be removed.

```typescript
enum Direction {
  Up = 'up',
  Down = 'down',
  Left = 'left',
  Right = 'right',
}
const directions = Object.values(Direction);

// Error: Element implicitly has an 'any' type because index expression is not of type 'number'.(7015)
// Only one of the two `as` assertions necessary to fix error, but neither are flagged as redundant.
for (const key of Object.keys(directions) as keyof directions[]) {
  const direction = directions[key as keyof typeof directions];
}
```

##### Type guards can be used to improve type inference

```typescript
function isSomeInterface(x: unknown): x is SomeInterface {
  return (
    'name' in x &&
    typeof x.name === 'string' &&
    'length' in x &&
    typeof x.length === 'number'
  );
}
```

🚫 Type assertion

```typescript
function f(x: SomeInterface | SomeOtherInterface) {
  console.log((x as SomeInterface).name);
}
```

✅ Narrowing with type guard

```typescript
function f(x: SomeInterface | SomeOtherInterface) {
  if (isSomeInterface(x)) {
    // Type of x: 'SomeInterface | SomeOtherInterface'
    console.log(x.name); // Type of x: 'SomeInterface'. Type of x.name: 'string'.
  }
}
```

```typescript
const nftMetadataResults = await Promise.allSettled(...);

nftMetadataResults
  .filter((promise) => promise.status === 'fulfilled')
  .forEach((elm) =>
    this.updateNft(
      elm.value.nft, // Property 'value' does not exist on type 'PromiseRejectedResult'.ts(2339)
      ...
    ),
  );
```

🚫 Type assertion

```typescript
(nftMetadataResults.filter(
    (promise) => promise.status === 'fulfilled',
  ) as { status: 'fulfilled'; value: NftUpdate }[])
  .forEach((elm) =>
    this.updateNft(
      elm.value.nft,
      ...
    ),
  );
```

✅ Use a type guard as the predicate for the filter operation, enabling TypeScript to narrow the filtered results to `PromiseFulfilledResult` at the type level

```typescript
nftMetadataResults.filter(
    (result): result is PromiseFulfilledResult<NftUpdate> =>
      result.status === 'fulfilled',
  )
  .forEach((elm) =>
    this.updateNft(
      elm.value.nft,
      ...
    ),
  );
```

#### Acceptable usages of `as`

##### To prevent or fix `any` usage

Unsafe as type assertions may be, they are still categorically preferable to using `any`.

- With type assertions, we still get working intellisense, autocomplete, and other IDE and compiler features using the asserted type.
- Type assertions also provide an indication of what the expected type is as intended by the author.
- For type assertions to an incompatible shape, use `as unknown as` as a last resort rather than `any` or `as any`.
- Often, the compiler will tell us exactly what the target type for an assertion needs to be, enabling us to avoid `as any`.
- Even an assertion to a wrong type still allows the compiler to show us warnings and errors as the code changes, and is therefore preferrable to the complete radio silence enforced by `any`.

```typescript
// Error: Argument of type '"getNftInformation"' is not assignable to parameter of type 'keyof NftController'.ts(2345)
// 'getNftInformation' is a private method of class 'NftController'
sinon.stub(nftController, 'getNftInformation');
```

🚫 `as any`

```typescript
sinon.stub(nftController, 'getNftInformation' as any);
```

✅ Compiler specifies that the target type should be `keyof NftController`

```typescript
sinon.stub(nftController, 'getNftInformation' as keyof typeof nftController);
```

##### For TypeScript syntax other than type assertion

- Writing type guards often reqiures using the `as` keyword.

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

- Key remapping in mapped types uses the `as` keyword.

```typescript
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties];
};
```

##### To type data objects whose shape and contents are determined at runtime, externally, or through deserialization

Preferably, this typing should be accompanied by runtime schema validation performed with type guards and unit tests.

- e.g. The output of `JSON.parse()` or `await response.json()` for a known JSON input.
- e.g. The type of a JSON file.

<!-- TODO: Add example -->

##### In tests, for mocking or to exclude irrelevant but required properties from an input object

<!-- TODO: Add examples -->

It's recommended to provide accurate typing if there's any chance that omitting properties affects the accuracy of the test.

If that is not the case, however, mocking only the properties needed in the test improves readability. It makes the intention and scope of the mock clear, and is much more convenient to write.

### Compiler Directives

<!-- TODO: Add section for `@ts-expect-error` -->

#### Avoid `any`

`any` is the most dangerous form of explicit type declaration, and should be completely avoided if possible.

- `any` doesn't represent the widest type, or indeed any type at all. `any` is a compiler directive for _disabling_ static type checking for the value or type to which it's assigned.
- `any` suppresses all error messages about its assignee. This includes errors that are changed or newly introduced by alterations to the code. This makes `any` the cause of dangerous **silent failures**, where the code fails at runtime but the compiler does not provide any prior warning.
- `any` subsumes all other types it comes into contact with. Any type that is in a union, intersection, is a property of, or has any other relationship with an `any` type or value is erased and becomes an `any` type itself.

```typescript
// Type of 'payload_0': 'any'
const handler:
  | ((payload_0: ComposableControllerState, payload_1: Patch[]) => void)
  | ((payload_0: any, payload_1: Patch[]) => void);
```
<!-- TODO: Add more examples: For instance, if you have a function that is declared to return any which actually returns an object, all properties of that object will be any. -->

- Because of this, `any` infects all surrounding and downstream code with its directive to suppress errors. This is the most dangerous characteristic of `any`, as it expands the surface area of unchecked code for which the TypeScript compiler can make no guarantees regarding type safety or runtime behavior.

##### Try `unknown` and `never` instead

###### `unknown`

- `unknown` is the universal supertype i.e. the widest possible type, equivalent to the universal set(U).
- Every type is assignable to `unknown`, but `unknown` is only assignable to `unknown`.
- When typing the _assignee_, `any` and `unknown` are completely interchangeable since every type is assignable to both.
- `any` usage is often motivated by a need to find a placeholder type that could be anything. `unknown` is the most likely type-safe substitute for `any` in these cases.

###### `never`

- `never` is the universal subtype i.e. the narrowest possible type, equivalent to the null set(∅).
- `never` is assignable to every type, but the only type that is assignable to `never` is `never`.
- When typing the _assigned_:
  - `unknown` is unable to replace `any`, as `unknown` is only assignable to `unknown`.
  - The type of the _assigned_ must be a subtype of the _assignee_.
  - `never` is worth trying, as it is the universal subtype and assignable to all types.

<!-- TODO: Add examples -->

##### Don't allow `any` to be used as a generic default argument

Some generic types use `any` as a default generic argument. This can silently introduce an `any` type into the code, causing unexpected behavior and suppressing useful errors.

🚫

```typescript
const NETWORKS = new Map({
  mainnet: '0x1',
  goerli: '0x5',
}); // Type 'Map<any, any>'

const mockGetNetworkConfigurationByNetworkClientId = jest.fn(); // Type 'jest.Mock<any, any>'
mockGetNetworkConfigurationByNetworkClientId.mockImplementation(
  (origin, type) => {},
); // No error!
// Even though 'mockImplementation' should only accept callbacks with a signature of '(networkClientId: string) => NetworkConfiguration | undefined'
```

✅

```typescript
const NETWORKS = new Map<string, `0x${string}`>({
  mainnet: '0x1',
  goerli: '0x5',
}); // Type 'Map<string, `0x${string}`>'

const mockGetNetworkConfigurationByNetworkClientId = jest.fn<
  ReturnType<NetworkController['getNetworkConfigurationByNetworkClientId']>,
  Parameters<NetworkController['getNetworkConfigurationByNetworkClientId']>
>(); // Type 'jest.Mock<NetworkConfiguration | undefined, [networkClientId: string]>'
mockGetNetworkConfigurationByNetworkClientId.mockImplementation(
  (origin, type) => {},
);
// Argument of type '(origin: any, type: any) => void' is not assignable to parameter of type '(networkClientId: string) => NetworkConfiguration | undefined'.
// Target signature provides too few arguments. Expected 2 or more, but got 1.ts(2345)
```

#### Acceptable usages of `any`

##### Assigning new properties to a generic type at runtime

In most type errors involving property access or runtime property assignment, `any` usage can be avoided by substituting with `as unknown as`.

🚫

```typescript
for (const key of getKnownPropertyNames(this.internalConfig)) {
  (this as any)[key] = this.internalConfig[key];
}

delete addressBook[chainId as any];
// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ [chainId: `0x${string}`]: { [address: string]: AddressBookEntry; }; }'.
//  No index signature with a parameter of type 'string' was found on type '{ [chainId: `0x${string}`]: { [address: string]: AddressBookEntry; }; }'.ts(7053)
```

✅

```typescript
for (const key of getKnownPropertyNames(this.internalConfig)) {
  (this as unknown as typeof this.internalConfig)[key] =
    this.internalConfig[key];
}

delete addressBook[chainId as unknown as `0x${string}`];
```

However, when assigning to a generic type, using `as any` is the only solution.

🚫

```typescript
(state as RateLimitState<RateLimitedApis>).requests[api][origin] = previous + 1;
// is generic and can only be indexed for reading.ts(2862)
```

✅

```typescript
(state as any).requests[api][origin] = previous + 1;
```

Even in this case, however, `any` usage might be avoidable by using `Object.assign` or spread operator syntax instead of assignment.

```typescript
Object.assign(state, {
  requests: {
    ...state.requests,
    [api]: { [origin] },
  },
});

{
  ...state,
  requests: {
    ...state.requests,
    [api]: { [origin] },
  },
};
```

##### Within generic constraints

✅

```typescript
class BaseController<
  ...,
  messenger extends RestrictedControllerMessenger<N, any, any, string, string>
> ...
```

- In general, using `any` in this context is not harmful in the same way that it is in other contexts, as the `any` types only are not directly assigned to any specific variable, and only function as constraints.
- That said, more specific constraints provide better type safety and intellisense, and should be preferred wherever possible.

##### Catching errors

- `catch` only accepts `any` and `unknown` as the error type.
- Recommended: Use `unknown` with type guards like `isJsonRpcError`.
- Avoid typing an error object with `any` if it is passed on to be used elsewhere instead of just being thrown, as the `any` type will infect the downstream code.

##### In tests, for mocking or to intentionally break features

<!-- TODO: Add examples. Should be clear why type assertions couldn't be used instead -->

## Functions

##### For functions and methods, provide explicit return types

Although TypeScript is capable of inferring return types, adding them explicitly makes it much easier for the reader to see the API from the code alone and prevents unexpected changes to the API from emerging.

🚫

```typescript
async function removeAccount(address: Hex) {
  const keyring = await this.getKeyringForAccount(address);

  if (!keyring.removeAccount) {
    throw new Error(KeyringControllerError.UnsupportedRemoveAccount);
  }
  keyring.removeAccount(address);
  this.emit('removedAccount', address);

  await this.persistAllKeyrings();
  return this.fullUpdate();
}
```

✅

```typescript
async function removeAccount(address: Hex): Promise<KeyringControllerState> {
  const keyring = await this.getKeyringForAccount(address);

  if (!keyring.removeAccount) {
    throw new Error(KeyringControllerError.UnsupportedRemoveAccount);
  }
  keyring.removeAccount(address);
  this.emit('removedAccount', address);

  await this.persistAllKeyrings();
  return this.fullUpdate();
}
```
