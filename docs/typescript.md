# TypeScript Guidelines

These guidelines specifically apply to TypeScript.

## Types

### Type Inference

TypeScript is very good at inferring types. A well-maintained codebase can provide strict type safety with explicit type annotations being the exception rather than the rule.

When writing TypeScript, function and class signatures, and types that express the domain model of the codebase will require explicit definitions and annotations.

However, for other types and values that are derived from those fundamentals, type inferences should generally be preferred over type declarations and assertions.

There are several reasons for this:

#### Advantages

- Explicit type annotations (`:`) and type assertions (`as`, `!`) prevent inference-based narrowing of the user-supplied types.
  - The compiler errs on the side of trusting user input, which prevents it from providing additional, and sometimes crucial, type information that it could otherwise infer.
  <!-- TODO: Expand into entry. Add example. -->
  - In TypeScript v4.9+ the `satisfies` operator can be used to assign type constraints that are narrowable through type inference.
- Type inferences are responsive to changes in code without requiring user input, while annotations and assertions rely on hard-coding, making them brittle against code drift.
- The `as const` operator can be used to further narrow an inferred abstract type into a specific literal type.

🚫 Type declarations

```ts
const name: string = 'METAMASK'; // Type 'string'

const BUILT_IN_NETWORKS = new Map<string, `0x${string}`>([
  ['mainnet', '0x1'],
  ['goerli', '0x5'],
]); // Type 'Map<string, `0x${string}`>'
```

✅ Type inferences

```ts
const name = 'METAMASK'; // Type 'METAMASK'

const BUILT_IN_NETWORKS = {
  mainnet: '0x1',
  goerli: '0x5',
} as const; // Type { readonly mainnet: '0x1'; readonly goerli: '0x5'; }
```

#### Example: To annotate or not to annotate

```ts
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

```ts
// Type 'TransactionMeta'
const updatedTransactionMeta: TransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected,
}; // resolves error
```

✅ Add `as const` and leave to inference

```ts
// Type narrower than 'TransactionMeta': { status: TransactionStatus.rejected; ... }
// (doesn't include 'error' property)
const updatedTransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected as const,
}; // resolves error
```

### Type Narrowing

There is a clear exception to the above: if an explicit type annotation or assertion can narrow an inferred type further, thereby improving its accuracy, it should be applied.

##### Avoid widening a type with a type annotation

> **Warning**<br />
> Double-check that a declared type is narrower than the inferred type.<br />
> Enforcing an even wider type defeats the purpose of adding an explicit type annotation, as it _loses_ type information instead of adding it.

🚫

```ts
const chainId: string = this.messagingSystem(
  'NetworkController:getProviderConfig',
).chainId; // Type 'string'
```

✅

```ts
const chainId = this.messagingSystem(
  'NetworkController:getProviderConfig',
).chainId; // Type '`0x${string}`'
```

##### When instantiating an empty container type, provide a type annotation

This is one case where type inference is unable to reach a useful conclusion without user-provided information. Since the compiler cannot arbitrarily restrict the range of types that could be inserted into the container, it has to assume the widest type, which is often `any`. It's up to the user to narrow that into the intended type with an explicit type annotation.

🚫

```ts
const tokens = []; // Type 'any[]'
const tokensMap = new Map(); // Type 'Map<any, any>'
```

✅

```ts
const tokens: string[] = []; // Type 'string[]'
const tokensMap = new Map<string, Token>(); // Type 'Map<string, Token>'
```

##### Type guards and null checks can be used to improve type inference

<!-- TODO: Add explanation and examples -->

```ts
function isSomeInterface(x: unknown): x is SomeInterface {
  return (
    'name' in x &&
    typeof x.name === 'string' &&
    'length' in x &&
    typeof x.length === 'number'
  );
}

function f(x: SomeInterface | SomeOtherInterface) {
  if (isSomeInterface(x)) {
    // Type of x: 'SomeInterface | SomeOtherInterface'
    console.log(x.name); // Type of x: 'SomeInterface'. Type of x.name: 'string'.
  }
}`
```

### Type Assertions

`as` assertions are unsafe. They overwrite type-checked and inferred types with user-supplied types that suppress compiler errors.

#### Avoid `as`

Type assertions make the code brittle against changes. While TypeScript will throw type errors against some unsafe or structurally unsound type assertions, it will generally accept the user-supplied type without type-checking. This can cause silent failures where errors are suppressed, even though the type's relationship to the rest of the code, or the type itself, has been altered so that the type assertion is no longer valid.

#### Acceptable usages of `as`

##### To prevent or fix `any` usage

Unsafe as type assertions may be, they are still categorically preferable to using `any`.

- With type assertions, we still get working intellisense, autocomplete, and other IDE features.
- Type assertions also provide an indication of the expected type as intended by the author.
- For type assertions to an incompatible shape, use `as unknown as` as a last resort rather than `any` or `as any`.

##### To define user-defined type guards

`as` syntax is often required to write type guards. See https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates.

##### To type data objects whose shape and contents are determined at runtime

Preferably, this typing should be accompanied by schema validation performed with type guards and unit tests.

- e.g. The output of `JSON.parse()` or `await response.json()` for a known JSON input.
- e.g. The type of a JSON file.

<!-- TODO: Add example -->

##### In tests, for mocking or to exclude irrelevant but required properties from an input object

<!-- TODO: Add examples -->

It's recommended to provide accurate typing if there's any chance that omitting properties affects the accuracy of the test.

Otherwise, only mocking the properties needed in the test improves readability by making the intention and scope of the mocking clear, not to mention being convenient to write.

### Compiler Directives

<!-- TODO: Add section for `@ts-expect-error` -->

#### Avoid `any`

`any` is the most dangerous form of explicit type declaration, and should be completely avoided if possible.

- `any` doesn't represent the widest type, or indeed any type at all. `any` is a compiler directive for _disabling_ static type checking for the value or type to which it's assigned.
- `any` suppresses all error messages about its assignee. This includes errors that are changed or newly introduced by alterations to the code. This makes `any` the cause of dangerous **silent failures**, where the code fails at runtime but the compiler does not provide any prior warning.
- `any` subsumes all other types it comes into contact with. Any type that is in a union, intersection, is a property of, or has any other relationship with an `any` type or value is erased and becomes an `any` type itself.

<!-- TODO: Add examples -->

```ts
const handler:
  | ((payload_0: ComposableControllerState, payload_1: Patch[]) => void)
  | ((payload_0: any, payload_1: Patch[]) => void); // Type of 'payload_0': 'any'
```

- `any` pollutes all surrounding and downstream code.
<!-- TODO: Add examples -->

#### Fixes for `any`

##### Try `unknown` instead

`unknown` is the universal supertype i.e. the widest possible type.

- When typing the assignee, `any` and `unknown` are interchangeable (every type is assignable to both).
- However, when typing the assigned, `unknown` can't be used to replace `any`, as `unknown` is only assignable to `unknown`. In this case, try `never`, which is assignable to all types.
<!-- TODO: Add example -->

##### Don't allow generic types to use `any` as a default argument

Some generic types use `any` as a default generic argument. This can silently introduce an `any` type into the code, causing unexpected behavior and suppressing useful errors.

🚫

```ts
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

```ts
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

```ts
for (const key of getKnownPropertyNames(this.internalConfig)) {
  (this as any)[key] = this.internalConfig[key];
}

delete addressBook[chainId as any];
// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ [chainId: `0x${string}`]: { [address: string]: AddressBookEntry; }; }'.
//  No index signature with a parameter of type 'string' was found on type '{ [chainId: `0x${string}`]: { [address: string]: AddressBookEntry; }; }'.ts(7053)
```

✅

```ts
for (const key of getKnownPropertyNames(this.config)) {
  (this as unknown as typeof this.config)[key] = this.config[key];
}

delete addressBook[chainId as unknown as `0x${string}`];
```

However, when assigning to a generic type, using `as any` is the only solution.

🚫

```ts
(state as RateLimitState<RateLimitedApis>).requests[api][origin] = previous + 1;
// is generic and can only be indexed for reading.ts(2862)
```

✅

```ts
(state as any).requests[api][origin] = previous + 1;
```

Even in this case, however, `any` usage might be avoidable by using `Object.assign` or spread operator syntax instead of assignment.

```ts
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

```ts
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

<!-- TODO: Add examples -->

## Functions

##### For functions and methods, provide explicit return types

Although TypeScript is capable of inferring return types, adding them explicitly makes it much easier for the reader to see the API from the code alone and prevents unexpected changes to the API from emerging.

🚫

```ts
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

```ts
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
