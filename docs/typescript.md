# TypeScript Guidelines

## Introduction

### Purpose

The _purpose_ of this document is to:

- Present best practices and preferred conventions for contributing type-safe, maintainable TypeScript code.
- Explain underlying concepts for issues that are not effectively resolved or prevented with blind compliance.
- Serve as a **reference** during code review to encourage preferred approaches, minimize redundant discussion, and educate contributors about established practices.

### Scope

The intended _scope_ of this document does **not** include:

- Becoming a comprehensive resource that covers all major aspects of syntax and style.
- Redundantly listing or explaining conventions that are enforceable with linters or rulesets.
- Endorsing practices for TypeScript code in external repos.

## Types

TypeScript provides a range of syntax for communicating type information with the compiler.

- The compiler performs **type inference** on all types and values in the code.
- The user can assign **type annotations** (`:`, `satisfies`) to override inferred types or add type constraints.
- The user can add **type assertions** (`as`, `!`) to force the compiler to accept user-supplied types even if they contradicts the inferred types.
- Finally, there are **escape hatches** that let type checking be disabled (`any`, `@ts-expect-error`) for a certain scope of code.

The order of this list represents the general order of preference for using these features.

### Type Inference

TypeScript is very good at inferring types. Explicit type annotations and assertions are the exception rather than the rule in a well-managed TypeScript codebase.

Some fundamental type information must always be supplied by the user, such as function and class signatures, interfaces for interacting with external entities or data types, and types that express the domain model of the codebase.

However, for most types, inference should be preferred over annotations and assertions.

##### Prefer type inference over annotations and assertions

- Explicit type annotations (`:`) and type assertions (`as`, `!`) prevent inference-based narrowing of the user-supplied types.
  - The compiler errs on the side of trusting user input, which prevents it from utilizing additional type information that it is able to infer.
  - The `satisfies` operator is an exception to this rule.
- Type inferences are responsive to changes in code without requiring user input, while annotations and assertions rely on hard-coding, making them brittle against code drift.
- The `as const` operator can be used to narrow an inferred abstract type into a specific literal type, or do the same for the elements of an array or object.

##### Avoid unintentionally widening an inferred type with a type annotation

Enforcing a wider type defeats the purpose of adding an explicit type declaration, as it _loses_ type information instead of adding it. Double-check that the declared type is narrower than the inferred type.

###### Example (aba42b65-1cb9-4df0-881e-c2e0e79db0bd)

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

###### Example (e9b0d703-032d-428b-a232-f5aa56a94470)

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

🚫

Adding a type annotation _does_ prevent the error above from being produced:

```typescript
// Type 'TransactionMeta'
const updatedTransactionMeta: TransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected,
};
```

✅

However, `TransactionMeta` is a [discriminated union](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions) of two separate types — "not failed" and "failed" — and the property that acts as the discriminator is `status`. Instead of using `TransactionMeta`, which specifies that a `error` property _could_ be present, it would be better to get TypeScript to infer the first of the two types ("not failed"), which guarantees that `error` is not present. We can do this by adding `as const` after `TransactionStatus.rejected`:

```typescript
const updatedTransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected as const,
};
```

### Type Annotations

An explicit type annotation is acceptable for overriding an inferred type if:

1. It can further narrow an inferred type, supplying type information that the compiler cannot infer or access otherwise.
2. It is being used to enforce a type constraint, not assign a type definition. For this use case, `satisfies` is preferred over `:`.

Compared to type assertions, type annotations are more responsive to code drift. If the assignee's type becomes incompatible with the assigned type annotation, the compiler will raise a type error, whereas in most cases a type assertion will still suppress the error.

##### Prefer the `satisfies` operator over the `:` operator for enforcing type constraints

Introduced in [TypeScript 4.9](https://devblogs.microsoft.com/typescript/announcing-typescript-4-9/), the `satisfies` operator can be used to enforce a type constraint, while also allowing the compiler to fully narrow the assigned type through inference.

###### Example (21ed5949-8d34-4754-b806-412de1696f46)

🚫 Use a type annotation for type validation

```typescript
const updatedTransactionMeta: TransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected,
};

updatedTransactionMeta.error; // Property 'error' does not exist on type '{ status: TransactionStatus.approved | TransactionStatus.cancelled | TransactionStatus.confirmed | TransactionStatus.dropped | TransactionStatus.rejected | TransactionStatus.signed | TransactionStatus.submitted | TransactionStatus.unapproved; ... }'.(2339)
```

✅ Use the `satisfies` operator for type validation

```typescript
const updatedTransactionMeta = {
  ...transactionMeta,
  status: TransactionStatus.rejected as const,
} satisfies TransactionMeta;

updatedTransactionMeta.error; // Property 'error' does not exist on type '{ status: TransactionStatus.rejected; ... }'.(2339)
```

##### When instantiating an empty composite data-type value, provide a type annotation

This is a special case where type inference cannot be expected to reach a useful conclusion without user-provided information.

The compiler doesn't have any values to use for inferring a type, and it cannot arbitrarily restrict the range of types that could be inserted into the collection. Given these restrictions, it has to assume the widest type, which is often `any`.

It's up to the user to appropriately narrow down this type by adding an explicit annotation that provides information about the user's intentions.

###### Example (b5a1175c-919f-4822-b92b-53a3d9dcd2e7)

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

##### When typing an extensible data type, use a type annotation

The reason type inference and the `satisfies` operator are generally preferred over type annotations is that they provide us with the narrowest applicable type signature.

When typing an extensible data type, however, this becomes a liability, because the narrowest type signature by definition doesn't include any newly assigned properties or elements. Therefore, when declaring or instantiating an object, array, or class, explicitly assign a type annotation, unless it is intended to be immutable.

###### Example (a5fc6e57-2609-41c2-8315-558824bfffed)

🚫 Type inference, `satisfies` operator

```typescript
// const SUPPORTED_CHAIN_IDS: ("0x1" | "0x38" | "0xa" | "0x2105" | "0x89" | "0xa86a" | "0xa4b1" | "0xaa36a7" | "0xe708")[]
export const SUPPORTED_CHAIN_IDS = [ // inference
  CHAIN_IDS.ARBITRUM,
  CHAIN_IDS.AVALANCHE,
  ...
  CHAIN_IDS.SEPOLIA,
];
export const SUPPORTED_CHAIN_IDS = [ // `satisfies` operator
  ...
] satisfies `0x${string}`[];

const { chainId } = networkController.state.providerConfig // Type of 'chainId': '`0x${string}`';
SUPPORTED_CHAIN_IDS.includes(chainId) // Argument of type '`0x${string}`' is not assignable to parameter of type '"0x1" | "0x38" | "0xa" | "0x2105" | "0x89" | "0xa86a" | "0xa4b1" | "0xaa36a7" | "0xe708"'.ts(2345)
```

✅ Type annotation

```typescript
export const SUPPORTED_CHAIN_IDS: `0x${string}`[] = [ // type annotation
  ...
];
const { chainId } = networkController.state.providerConfig // Type of 'chainId': '`0x${string}`';
SUPPORTED_CHAIN_IDS.includes(chainId) // No error
```

### Type Assertions

`as` assertions are inherently unsafe. They overwrite type-checked and inferred types with user-supplied types that suppress compiler errors.

They should only be introduced into the code if the accurate type is unreachable through other means.

##### Document safe or necessary use of type assertions

When a type assertion is absolutely necessary due to constraints or is even safe due to runtime checks, we should document the reasoning behind its usage in the form of a comment.

<!-- TODO: Add example -->

#### Avoid `as`

Type assertions make the code brittle against changes.

While TypeScript and ESLint will flag some unsafe, structurally unsound, or redundant type assertions, they will generally accept user-supplied types without further type-checking.

This can cause silent failures or false negatives where errors are suppressed. This is especially damaging as the codebase accumulates changes over time. Type assertions may continue to silence errors, even though the type itself, or the type's relationship to the rest of the code may have been altered so that the asserted type is no longer valid.

##### Redundant or unnecessary `as` assertions are not flagged for removal

Type assertions can also cause false positives, because assertions are independent expressions, untied to the type errors they were intended to fix. Even if code drift fixes or removes a particular type error, the type assertions that were put in place to fix that error will provide no indication that they are no longer necessary and now should be removed.

###### Example (3675ab71-bcd6-4325-ac18-8ba4dd8ec03c)

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
for (const key of Object.keys(directions) as (keyof typeof directions)[]) {
  const direction = directions[key as keyof typeof directions];
}
```

##### Type guards can be used to improve type inference and avoid type assertion

###### Example (50c3fbc9-c2d7-4140-9f75-be5f0a56d541)

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
  if (x.name) {
    // We know that `x` is `SomeInterface` because `x` has a `name` but `SomeOtherInterface` does not
    console.log((x as SomeInterface).name);
  }
}
```

✅ Narrowing with type guard

```typescript
function f(x: SomeInterface | SomeOtherInterface) {
  if (isSomeInterface(x)) {
    console.log(x.name); // Type of x: 'SomeInterface'. Type of x.name: 'string'.
  }
}
```

###### Example (f7ff4b0d-e5e9-4568-b916-5153ddd2095b)

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

> Note: The [`is` assertion is unnecessary](https://github.com/microsoft/TypeScript/pull/57465) as of TypeScript v5.5.

#### Acceptable usages of `as`

Although `as` is dangerous and discouraged from being used, the following is not intended as an exhaustive list of its valid use cases.

##### `as` is acceptable to use for preventing or fixing `any` usage

Type assertions are unsafe, but they are still always preferred to introducing `any` into the code.

- With type assertions, we still get working intellisense, autocomplete, and other IDE and compiler features using the asserted type.
- Type assertions also provide an indication of what the author intends or expects the type to be.
- Often, the compiler will tell us exactly what the target type for an assertion needs to be, enabling us to avoid `as any`.

###### Example (2ee8f56a-e3be-417b-a2c0-260c1319b755)

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

- Even an assertion to a wrong type still allows the compiler to show us warnings and errors as the code changes, and is therefore preferrable to the dangerous radio silence enforced by `any`.
- For type assertions to an incompatible shape, use `as unknown as` as a last resort rather than `any` or `as any`.
<!-- TODO: Add example -->

##### `as` is always acceptable to use for TypeScript syntax other than type assertion

- Writing type guards often requires using the `as` keyword.

###### Example (f16df571-266e-4030-b002-49554558ccd7)

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

- Key remapping in mapped types uses the `as` keyword.

###### Example (6ffd8c99-4768-42e1-8cb7-5710d14f8552)

```typescript
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties];
};
```

##### `as` is acceptable to use for typing data objects whose shape and contents are determined at runtime, externally, or through deserialization

Preferably, this typing should be accompanied by runtime schema validation performed with type guards and unit tests.

- e.g. The output of `JSON.parse()` or `await response.json()` for a known JSON input.
- e.g. The type of a JSON file.

<!-- TODO: Add example -->

##### `as` may be acceptable to use in tests, for mocking or to exclude irrelevant but required properties from an input object

It's recommended to provide accurate typing if there's any chance that omitting properties affects the accuracy of the test.

If that is not the case, however, mocking only the properties needed in the test improves readability. It makes the intention and scope of the mock clear, and is more convenient to write, which can encourage test writing and improve test coverage.

<!-- TODO: Add examples -->

### Escape Hatches

TypeScript provides several escape hatches that disable compiler type checks altogether and suppress compiler errors. Using these to ignore typing issues is dangerous and reduces the effectiveness of TypeScript.

- `@ts-expect-error`
  - Applies to a single line, which may contain multiple variables and errors.
  - Is allowed by the ESLint rule `@typescript-eslint/ban-ts-comment`, but is required to be accompanied by an explanation comment.
  - **It alerts users if an error it was suppressing has been resolved by changes in the code**:
    > **Error:** Unused '@ts-expect-error' directive.
- `as any`

  - Applies to a single instance of a single variable.
  - Is banned by the ESLint rule `@typescript-eslint/no-explicit-any.

- `@ts-ignore`

  - Applies to a line or block of code, which may contain multiple variables and errors.
  - Is banned by the ESLint rule `@typescript-eslint/ban-ts-comment`.

- `any`:
  - Applies to all instances of the target variable or type throughout the entire codebase, and in downstream code as well.
  - Is banned by the ESLint rule `@typescript-eslint/no-explicit-any.
  - Has the same effect as applying `@ts-ignore` to every single instance of the target variable or type.

#### Acceptable usages of `@ts-expect-error`

In general, `@ts-expect-error` usage should be reserved to situations where an error is the intended or expected result of an operation, not to silence errors when the correct typing solution couldn't be found.

##### Use `@ts-expect-error` to force runtime execution of a branch for validation or testing

Sometimes, there is a need to force a branch to execute at runtime for security or testing purposes, when that branch has correctly been inferred as being inaccessible by the TypeScript compiler.

###### Example (76b145a7-89bf-4f19-914b-d1c02e2db185)

✅

> **Error:** This comparison appears to be unintentional because the types '`0x${string}`' and '"**proto**"' have no overlap.ts(2367)

```typescript
exampleFunction(chainId: `0x${string}`) {
    // @ts-expect-error Suppressing to perform runtime check
    if (chainId === '__proto__') {
      return;
    }
    ...
}
```

##### `@ts-expect-error` may be acceptable to use in tests, to intentionally break features

###### Example (e299e95d-1c41-4251-85b6-f8064b22f577)

✅

```typescript
// @ts-expect-error Suppressing to test runtime error handling

// @ts-expect-error Intentionally testing invalid state

// @ts-expect-error We are intentionally passing bad input.
```

##### If accompanied by a TODO comment, `@ts-expect-error` is acceptable to use for marking errors that have clear plans of being resolved

<!-- TODO: Add example -->

#### Avoid `any`

`any` is the most dangerous form of explicit type declaration, and should be completely avoided.

Unfortunately, `any` is also such a tempting escape hatch from the TypeScript type system that there's a strong incentive to use it whenever a nontrivial typing issue is encountered. Entire teams can easily be enticed into a pattern of unblocking feature development by introducing `any` with the intention of fixing it later. This is a major source of tech debt, and its destructive influence on the type safety of a codebase cannot be understated.

Therefore, to prevent new `any` instances from being introduced into our codebase, it is not enough to rely on the `@typescript-eslint/no-explicit-any` ESLint rule. It's also necessary for all contributors to understand exactly why `any` is dangerous, and how it can be avoided.

The key thing to remember about `any` is that it does not resolve errors, but only hides them. The errors still affect the code, while `any` makes it impossible to assess and counteract their influence.

- `any` doesn't represent the widest type, or indeed any type at all. `any` is a compiler directive for _disabling_ static type checking for the value or type to which it's assigned.
- `any` suppresses all error messages about its assignee. This makes code with `any` usage brittle against changes, since the compiler is unable to update its feedback even when the code has changed enough to alter or remove the error, or even add new type errors.
- `any` subsumes all other types it comes into contact with. Any type that is in a union, intersection, is a property of, or has any other relationship with an `any` type or value becomes an `any` type itself. This represents an unmitigated loss of type information.

###### Example (1fb5b0ad-61a9-4ad8-9d84-e29b78d88325)

```typescript
// Type of 'payload_0': 'any'
const handler:
  | ((payload_0: ComposableControllerState, payload_1: Patch[]) => void)
  | ((payload_0: any, payload_1: Patch[]) => void);

function returnsAny(): any {
  return { a: 1, b: true, c: 'c' };
}
// Types of a, b, c are all `any`
const { a, b, c } = returnsAny();
```

- `any` infects all surrounding and downstream code with its directive to suppress errors. This is the most dangerous characteristic of `any`, as it causes the encroachment of unsafe code with no guarantees about type safety or runtime behavior.

All of this makes `any` a prominent cause of dangerous **silent failures** (false negatives), where the code fails at runtime but the compiler does not provide any prior warning, which defeats the purpose of using a statically-typed language.

##### If `any` is being used as the _assignee_ type, use `unknown` instead

`any` usage is often motivated by a need to find a placeholder type that could be anything. `unknown` is the most likely type-safe substitute for `any` in these cases.

- `unknown` is the universal supertype i.e. the widest possible type, equivalent to the universal set(U).
- Every type is assignable to `unknown`, but `unknown` is only assignable to `unknown`.
- When typing the _assignee_, `any` and `unknown` are completely interchangeable since every type is assignable to both.

##### If `any` is being used as the _assigned_ type, find an appropriate subtype of the _assignee_ type

Unfortunately, when typing the _assigned_ type, `unknown` cannot substitute `any` in most cases, because:

- `unknown` is only assignable to `unknown`.
- The type of the _assigned_ must be a subtype of the _assignee_, but `unknown` can only be a subtype of `unknown`.

###### Example (56165606-17db-479d-a2f7-cc95250f2129)

```typescript
function f1(arg1: string) { ... }
function f2(arg2: any) {
  f1(arg2) // `arg1` is the assignee type, and `arg2` is the assigned type.
}
```

🚫 `unknown`

```typescript
function f1(arg1: string) { ... }
function f2(arg2: unknown) {
  f1(arg2) // Error: Argument of type 'unknown' is not assignable to parameter of type 'string'.(2345)
}
```

✅ Subtype of `string`, the assignee type

```typescript
function f1(arg1: string) { ... }
function f2(arg2: `0x${string}`) {
  f1(arg2)
}
```

##### Don't allow `any` to be used as a generic default argument

Some generic types use `any` as a default generic argument. This can silently introduce an `any` type into the code, causing unexpected behavior and suppressing useful errors.

###### Example (c64ed0da-01f1-4b61-a28a-ff8e8ab3c8b5)

🚫

```typescript
const mockGetNetworkConfigurationByNetworkClientId = jest.fn(); // Type 'jest.Mock<any, any>'
mockGetNetworkConfigurationByNetworkClientId.mockImplementation(
  (origin, type) => {},
); // No error!
// Even though 'mockImplementation' should only accept callbacks with a signature of '(networkClientId: string) => NetworkConfiguration | undefined'
```

✅

```typescript
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

##### Prefer `as unknown as` over `as any`

In most type errors involving property access or runtime property assignment, `any` usage can be avoided by substituting with `as unknown as`.

###### Example (03d4fc8b-73a3-478a-a986-df89c9b80775)

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

#### Acceptable usages of `any`

##### `any` may be acceptable to use within generic constraints

###### Example (706045b1-1f01-4e24-ae02-d9a3a8e81615)

✅

```typescript
class BaseController<
  ...,
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  messenger extends RestrictedControllerMessenger<N, any, any, string, string>
> ...
```

- In general, using `any` in this context is not harmful in the same way that it is in other contexts, as the `any` types only are not directly assigned to any specific variable, and only function as constraints.
- More specific constraints provide better type safety and intellisense, and should be preferred wherever possible.
- This only applies to generic _constraints_. It does not apply to passing in `any` as a generic _argument_.

###### Example (7b9781b4-0f33-4619-ba50-a90b2594e23f)

🚫

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const controllerMessenger = ControllerMessenger<any, any>;
```

## Functions

##### For functions and methods, provide explicit return types

Although TypeScript is capable of inferring return types, adding them explicitly makes it much easier for the reader to see the API from the code alone and prevents unexpected changes to the API from emerging.

###### Example (a88b18ef-b066-4aa7-8106-bc244298f9e6)

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
