# TypeScript Guidelines

## Introduction

This is a collection of TypeScript best practices and preferred conventions for contributors to the _MetaMask_ project.

This document is not intended as a stand-in for linters or formatters. Emphasis is put on discussing underlying concepts and rationale, rather than listing rules and restrictions.

Type safety is the highest priority in these guidelines, even if that sometimes means recommending unconventional or opinionated practices.

Note that this document assumes that the reader has a high level of familiarity with TypeScript, and may omit explanations.

## Types

TypeScript provides a range of syntax for communicating type information with the compiler.

- The compiler performs **type inference** on all types and values in the code.
- The user can assign **type annotations** (`satisfies`, `:`) to override inferred types or add type constraints.
- The user can add **type assertions** (`!`, `as`) to force the compiler to accept user-supplied types even if they contradicts the inferred types.
- Finally, there are **escape hatches** that let type checking be disabled (`@ts-expect-error`, `any`) for a certain scope of code.

The order of this list represents the general order of preference for using these features.

### Type Inference

TypeScript is very good at inferring types. Explicit type annotations and assertions are the exception rather than the rule in a well-managed TypeScript codebase.

Some fundamental type information must always be supplied by the user, such as function and class signatures, interfaces for interacting with external entities or data types, and types that express the domain model of the codebase.

However, for most types, inference should be preferred over annotations and assertions.

#### Prefer type inference over annotations and assertions

- Explicit type annotations (`:`) and type assertions (`as`, `!`) prevent inference-based narrowing of the user-supplied types.
  - The compiler errs on the side of trusting user input, which prevents it from utilizing additional type information that it is able to infer.
  - The `satisfies` operator is an exception to this rule.
- Type inferences are responsive to changes in code without requiring user input, while annotations and assertions rely on hard-coding, making them brittle against code drift.
- The `as const` operator can be used to narrow an inferred abstract type into a specific literal type, or do the same for the elements of an array or object.

#### Avoid unintentionally widening an inferred type with a type annotation

Enforcing a wider type defeats the purpose of adding an explicit type declaration, as it _loses_ type information instead of adding it. Double-check that the declared type is narrower than the inferred type.

**Example <a id="example-aba42b65-1cb9-4df0-881e-c2e0e79db0bd"></a> ([🔗 permalink](#example-aba42b65-1cb9-4df0-881e-c2e0e79db0bd)):**

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

**Example <a id="example-e9b0d703-032d-428b-a232-f5aa56a94470"></a> ([🔗 permalink](#example-e9b0d703-032d-428b-a232-f5aa56a94470)):**

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

#### Prefer `satisfies` annotation over `:` annotation for enforcing type constraints

Introduced in [TypeScript 4.9](https://devblogs.microsoft.com/typescript/announcing-typescript-4-9/), the `satisfies` operator can be used to enforce a type constraint, while also allowing the compiler to fully narrow the assigned type through inference.

**Example <a id="example-21ed5949-8d34-4754-b806-412de1696f46"></a> ([🔗 permalink](#example-21ed5949-8d34-4754-b806-412de1696f46)):**

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

#### Provide a `:` annotation when instantiating an empty composite data-type value

This is a special case where type inference cannot be expected to reach a useful conclusion without user-provided information.

The compiler doesn't have any values to use for inferring a type, and it cannot arbitrarily restrict the range of types that could be inserted into the collection. Given these restrictions, it has to assume the widest type, which is often `any`.

It's up to the user to appropriately narrow down this type by adding an explicit annotation that provides information about the user's intentions.

**Example <a id="example-b5a1175c-919f-4822-b92b-53a3d9dcd2e7"></a> ([🔗 permalink](#example-b5a1175c-919f-4822-b92b-53a3d9dcd2e7)):**

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

#### Prefer a `:` annotation over `satisfies` when typing an extensible data type

The reason type inference and the `satisfies` operator are generally preferred over type annotations is that they provide us with the narrowest applicable type signature.

When typing an extensible data type, however, this becomes a liability, because the narrowest type signature by definition doesn't include any newly assigned properties or elements. Therefore, when declaring or instantiating an object, array, or class, explicitly assign a type annotation, unless it is intended to be immutable.

**Example <a id="example-a5fc6e57-2609-41c2-8315-558824bfffed"></a> ([🔗 permalink](#example-a5fc6e57-2609-41c2-8315-558824bfffed)):**

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

Type assertions are inherently unsafe and should only be used if the accurate type is unreachable through other means.

- Type assertions overwrite type-checked and compiler-inferred types with user-supplied types, and suppress the resulting compiler errors.
- Because type assertions do not affect runtime code in any way, it is likely that types asserted at compile time will conflict with values at runtime.
- Type assertions make the codebase brittle against changes.

  - As changes accumulate in the codebase, type assertions may continue to enforce type assignments that have become incorrect, or keep silencing errors that have changed. This can cause dangerous silent failures (false negatives).
  - Type assertions will also provide no indication when they become unnecessary or redundant (false positives). Even if changes in the code resolve the error that the assertion was silencing, TypeScript will keep the assertion in place.

    **Example <a id="example-3675ab71-bcd6-4325-ac18-8ba4dd8ec03c"></a> ([🔗 permalink](#example-3675ab71-bcd6-4325-ac18-8ba4dd8ec03c)):**

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

#### Document safe or necessary use of type assertions in comments

When a type assertion is used with a clear rationale, we should document the reasoning behind its usage in the form of a comment.

- A type assertion may be necessary to satisfy constraints, or to align with a type which is verified to be accurate by an external source of truth.

  **Example <a id="example-05558b5e-527e-46b0-8b40-918f28d05156"></a> ([🔗 permalink](#example-05558b5e-527e-46b0-8b40-918f28d05156)):**

  ✅

  ```typescript
  import contractMap from '@metamask/contract-metadata';

  type LegacyToken = {
    name: string;
    logo: `${string}.svg`;
    symbol: string;
    decimals: number;
    erc20?: boolean;
    erc721?: boolean;
  };

  export const STATIC_MAINNET_TOKEN_LIST = Object.entries(
    // This type assertion is to the known schema of the JSON object `contract-metadata.json`.
    contractMap as Record<Hex, LegacyToken>,
  ).reduce((acc, [base, contract]) => {
    const { name, symbol, decimals, logo, erc721 } = contract;
    return {
      ...acc,
      [base.toLowerCase()]: {
        ...{ name, symbol, decimals },
        address: base.toLowerCase(),
        iconUrl: `images/contract/${logo}`,
        isERC721: erc721 ?? false
        aggregators: [],
      },
    };
  }, {});
  ```

- A type assertion may be safe if it's supported by runtime validations.

  **Example <a id="example-81737669-75fb-46d6-b2ce-c09acd5b89ab"></a> ([🔗 permalink](#example-81737669-75fb-46d6-b2ce-c09acd5b89ab)):**

  ```typescript
  handle<Params extends JsonRpcParams, Result extends Json>(
    request: JsonRpcRequest<Params>,
    callback: (error: unknown, response: JsonRpcResponse<Result>) => void,
  ): void;

  handle<Params extends JsonRpcParams, Result extends Json>(
    requests: (JsonRpcRequest<Params> | JsonRpcNotification<Params>)[],
    callback: (error: unknown, responses: JsonRpcResponse<Result>[]) => void,
  ): void;

  handle<Params extends JsonRpcParams, Result extends Json>(
    requests: (JsonRpcRequest<Params> | JsonRpcNotification<Params>)[],
  ): Promise<JsonRpcResponse<Result>[]>;
  ```

  ✅

  ```typescript
  handle(
    req:
      | (JsonRpcRequest | JsonRpcNotification)[]
      | JsonRpcRequest
      | JsonRpcNotification,
    callback?: (error: unknown, response: never) => void,
  ) {
    ...
    if (Array.isArray(req) && callback) {
      return this.#handleBatch(
        req,
        // This assertion is safe because of the runtime checks validating that `req` is an array and `callback` is defined.
        // There is only one overload signature that satisfies both conditions, and its `callback` type is the one that's being asserted.
        callback as (
          error: unknown,
          responses?: JsonRpcResponse<Json>[],
        ) => void,
      );
    }
    ...
  }
  ```

- Rarely, a type assertion may be necessary to resolve or suppress a type error caused by a bug or limitation of an external library, or even the TypeScript language itself.

  **Example <a id="example-3d77c689-2bec-48d6-8c2e-c38b7b2079ea"></a> ([🔗 permalink](#example-3d77c689-2bec-48d6-8c2e-c38b7b2079ea)):**

  ✅

  ```typescript
  import { produceWithPatches } from 'immer';

  protected update(
    callback: (state: Draft<ControllerState>) => void | ControllerState,
  ): {
    nextState: ControllerState;
    patches: Patch[];
    inversePatches: Patch[];
  } {
    // We run into ts2589, "infinite type depth", if we don't assert `produceWithPatches` here.
    const [nextState, patches, inversePatches] = (
      produceWithPatches as unknown as (
        state: ControllerState,
        cb: typeof callback,
      ) => [ControllerState, Patch[], Patch[]]
    )(this.#internalState, callback);
    ...
  }
  ```

#### Avoid `as` assertions by using type guards to improve type inference

**Example <a id="example-50c3fbc9-c2d7-4140-9f75-be5f0a56d541"></a> ([🔗 permalink](#example-50c3fbc9-c2d7-4140-9f75-be5f0a56d541)):**

```typescript
type SomeInterface = { name: string; length: number };
type SomeOtherInterface = { value: boolean };

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
    console.log(x.name); // Type of x: 'SomeInterface'. Type of x.name: 'string'.
  }
}
```

**Example <a id="example-f7ff4b0d-e5e9-4568-b916-5153ddd2095b"></a> ([🔗 permalink](#example-f7ff4b0d-e5e9-4568-b916-5153ddd2095b)):**

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

> Note: The `is` type predicate in this example [is unnecessary as of TypeScript v5.5](https://github.com/microsoft/TypeScript/pull/57465).

#### Determine the target type for an `as` assertion by examining compiler error messages

Often, the compiler will tell us exactly what the target type for an assertion needs to be.

**Example <a id="example-2ee8f56a-e3be-417b-a2c0-260c1319b755"></a> ([🔗 permalink](#example-2ee8f56a-e3be-417b-a2c0-260c1319b755)):**

🚫 Compiler specifies that the target type should be `keyof NftController`

```typescript
// Error: Argument of type '"getNftInformation"' is not assignable to parameter of type 'keyof NftController'.ts(2345)
// 'getNftInformation' is a private method of class 'NftController'
sinon.stub(nftController, 'getNftInformation');
```

✅ `as` assertion to type specified by compiler

```typescript
sinon.stub(nftController, 'getNftInformation' as keyof typeof nftController);
```

#### `as` is always acceptable to use in TypeScript syntax that does not involve type assertions

- `as const` assertions.

- Key remapping in mapped types uses the `as` keyword.

  **Example <a id="example-6ffd8c99-4768-42e1-8cb7-5710d14f8552"></a> ([🔗 permalink](#example-6ffd8c99-4768-42e1-8cb7-5710d14f8552)):**

  ```typescript
  type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties];
  };
  ```

### Escape Hatches

TypeScript provides several escape hatches that disable compiler type checks altogether and suppress compiler errors. Using these to ignore typing issues is dangerous and reduces the effectiveness of TypeScript.

- `@ts-expect-error`
  - Applies to a single line, which may contain multiple variables and errors.
  - **It alerts users if an error it was suppressing is resolved by changes in the code:**
    > **Error:** Unused '@ts-expect-error' directive.
  - `@ts-expect-error` usage should generally be reserved to situations where an error is the intended or expected result of an operation, not to silence errors when the correct typing solution is difficult to find.
- `as any`

  - Applies only to a single instance of a single variable without propagating to other instances.

- `@ts-ignore`

  - Applies to a line or block of code, which may contain multiple variables and errors.
  - Does not propagate to instances of the target variable or type that are outside of its scope.

- `any`:
  - Applies to all instances of the target variable or type throughout the entire codebase, and in downstream code as well.
  - Has the same effect as applying `@ts-ignore` to every single instance of the target variable or type.

#### Use `@ts-expect-error` to force runtime execution of a branch for validation or testing

Sometimes, there is a need to force a branch to execute at runtime for security or testing purposes, even though that branch has correctly been inferred as being inaccessible by the TypeScript compiler.

This is often the case when downstream consumers of the code are using JavaScript and do not have access to compile-time guardrails.

**Example <a id="example-76b145a7-89bf-4f19-914b-d1c02e2db185"></a> ([🔗 permalink](#example-76b145a7-89bf-4f19-914b-d1c02e2db185)):**

🚫

> **Error:** This comparison appears to be unintentional because the types '\`0x${string}\`' and '"\_\_proto\_\_"' have no overlap.ts(2367)

```typescript
function exampleFunction(chainId: `0x${string}`) {
    if (chainId === '__proto__') {
      return;
    }
    ...
}
```

🚫

> **Error:** Argument of type '"\_\_proto\_\_"' is not assignable to parameter of type '\`0x${string}\`'.ts(2345)

```typescript
exampleFunction('__proto__');
```

✅

```typescript
function exampleFunction(chainId: `0x${string}`) {
    // @ts-expect-error Suppressing to perform runtime check
    if (chainId === '__proto__') {
      return;
    }
    ...
}
```

✅

```typescript
// @ts-expect-error Suppressing to perform runtime check
exampleFunction('__proto__');
```

#### `@ts-expect-error` may be acceptable to use in tests, to intentionally break features

**Example <a id="example-e299e95d-1c41-4251-85b6-f8064b22f577"></a> ([🔗 permalink](#example-e299e95d-1c41-4251-85b6-f8064b22f577)):**

✅

```typescript
// @ts-expect-error Suppressing to test runtime error handling

// @ts-expect-error Intentionally testing invalid state

// @ts-expect-error We are intentionally passing bad input.
```

#### If accompanied by a TODO comment, `@ts-expect-error` is acceptable to use for marking errors that have clear plans of being resolved

**Example <a id="example-43313247-4393-4966-b78e-378f636fedec"></a> ([🔗 permalink](#example-43313247-4393-4966-b78e-378f636fedec)):**

✅

```typescript
// @ts-expect-error TODO: remove this annotation once the `Eip1193Provider` class is released, resolving thi provider misalignment issue.
return new Web3Provider(provider);

// TODO: Fix this by handling or eliminating the undefined case
// @ts-expect-error This variable can be `undefined`, which would break here.
```

This recommendation applies to any disruptive change that creates many errors at once (e.g. dependency update, upstream refactor, package migration).

See [this entry](https://github.com/MetaMask/core/blob/main/docs/package-migration-process-guide.md#4-resolve-or-todo-downstream-errors) in the core repo "package migration process guide," which recommends that complex or blocked errors should be annotated with a `// @ts-expect-error TODO:` comment, and then revisited once the disruptive change has been completed.

#### Always avoid `any`

`any` is the most dangerous form of explicit type declaration, and should be completely avoided.

Unfortunately, `any` is also such a tempting escape hatch from the TypeScript type system that there's a strong incentive to use it whenever a nontrivial typing issue is encountered. Entire teams can easily be enticed into a pattern of unblocking feature development by introducing `any` with the intention of fixing it later. This is a major source of tech debt, and its destructive influence on the type safety of a codebase cannot be understated.

Therefore, to prevent new `any` instances from being introduced into our codebase, it is not enough to rely on the `@typescript-eslint/no-explicit-any` ESLint rule. It's also necessary for all contributors to understand exactly why `any` is dangerous, and how it can be avoided.

The key thing to remember about `any` is that it does not resolve errors, but only hides them. The errors still affect the code, while `any` makes it impossible to assess and counteract their influence.

- `any` doesn't represent the widest type, or indeed any type at all. `any` is a compiler directive for _disabling_ static type checking for the value or type to which it's assigned.
- `any` suppresses all error messages about its assignee. This makes code with `any` usage brittle against changes, since the compiler is unable to update its feedback even when the code has changed enough to alter or remove the error, or even add new type errors.
- `any` subsumes all other types it comes into contact with. Any type that is in a union, intersection, is a property of, or has any other relationship with an `any` type or value becomes an `any` type itself. This represents an unmitigated loss of type information.

  **Example <a id="example-1fb5b0ad-61a9-4ad8-9d84-e29b78d88325"></a> ([🔗 permalink](#example-1fb5b0ad-61a9-4ad8-9d84-e29b78d88325)):**

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

#### Prefer type assertions over `any`

Type assertions are unsafe, but they are still always preferred over introducing `any` into the code.

- With type assertions, we still get working intellisense, autocomplete, and other IDE and compiler features using the asserted type.
- Type assertions also provide an indication of what the author intends or expects the type to be.
- Even an assertion to a wrong type still allows the compiler to show us warnings and errors as the code changes.

#### If `any` is being used as the _assignee_ type, try `unknown` first, and then try narrowing to an appropriate supertype of the _assigned_ type

`any` usage is often motivated by a need to find a placeholder type that could be anything. `unknown` is a likely type-safe substitute for `any` in these cases.

- `unknown` is the universal supertype i.e. the widest possible type, equivalent to the universal set(U).
- Every type is assignable to `unknown`, but `unknown` is not assignable to any type but itself.
- When typing the _assignee_, `any` and `unknown` are completely interchangeable since every type is assignable to both.

<!-- TODO: Add example -->

#### If `any` is being used as the _assigned_ type, try `never` first, and then try widening to an appropriate subtype of the _assignee_ type

Unfortunately, when typing the _assigned_ type, `unknown` cannot substitute `any` in most cases, because:

- `unknown` is only assignable to `unknown`.
- The type of the _assigned_ must be a subtype of the _assignee_, but `unknown` can only be a subtype of `unknown`.

However, `never` is assignable to all types

> **Note:** Once `unknown` has been ruled out as a substitute for `any`, trying `never` serves as a valuable test: It tells us that the search space for the correct substitute type is bounded to subtypes of the _assignee_ type.

**Example <a id="example-56165606-17db-479d-a2f7-cc95250f2129"></a> ([🔗 permalink](#example-56165606-17db-479d-a2f7-cc95250f2129)):**

```typescript
function f1(arg1: string) { ... }
```

🚫 `any`

In the function call `f1(arg2)`, the argument `arg2` is the _assigned_ type and the parameter `arg1` is the _assignee_ type.

```typescript
function f2(arg2: any) {
  f1(arg2);
}
```

🚫 `unknown`

> **Error:** Argument of type 'unknown' is not assignable to parameter of type 'string'.(2345)

```typescript
function f2(arg2: unknown) {
  f1(arg2); // Error
}
```

✅ `never`

This works, but `arg2` can be widened further.

```typescript
function f2(arg2: never) {
  f1(arg2); // No error
}
```

✅ Subtype of `string`, the assignee type

```typescript
function f2(arg2: `0x${string}`) {
  f1(arg2); // No error
}
```

#### Don't allow generic type parameters to resolve to a default type of `any`

Some generic types use `any` as a generic parameter default. If not consciously avoided, this can silently introduce an `any` type into the code, causing unexpected behavior and suppressing useful errors.

**Example <a id="example-c64ed0da-01f1-4b61-a28a-ff8e8ab3c8b5"></a> ([🔗 permalink](#example-c64ed0da-01f1-4b61-a28a-ff8e8ab3c8b5)):**

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

#### Prefer `as unknown as` over `as any`

In most type errors involving property access or runtime property assignment, `any` usage can be avoided by substituting with `as unknown as`.

**Example <a id="example-03d4fc8b-73a3-478a-a986-df89c9b80775"></a> ([🔗 permalink](#example-03d4fc8b-73a3-478a-a986-df89c9b80775)):**

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

#### `any` may be acceptable to use within generic constraints

**Example <a id="example-706045b1-1f01-4e24-ae02-d9a3a8e81615"></a> ([🔗 permalink](#example-706045b1-1f01-4e24-ae02-d9a3a8e81615)):**

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

  **Example <a id="example-7b9781b4-0f33-4619-ba50-a90b2594e23f"></a> ([🔗 permalink](#example-7b9781b4-0f33-4619-ba50-a90b2594e23f)):**

  🚫

  ```typescript
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  const controllerMessenger = ControllerMessenger<any, any>;
  ```

## Functions

#### For functions and methods, provide explicit return types

Although TypeScript is capable of inferring return types, adding them explicitly makes it much easier for the reader to see the API from the code alone and prevents unexpected changes to the API from emerging.

**Example <a id="example-a88b18ef-b066-4aa7-8106-bc244298f9e6"></a> ([🔗 permalink](#example-a88b18ef-b066-4aa7-8106-bc244298f9e6)):**

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
