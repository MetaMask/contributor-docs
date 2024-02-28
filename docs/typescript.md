# **Typescript guidelines**

***Purpose***: Establish best practices for writing robust, maintainable, and efficient TypeScript code.

***Audience***: Internal and external developers using TypeScript in Metamask.


## **Interfaces**
By defining expected object structures with interfaces, TypeScript promotes developer intent and unlocks powerful IDE features like accurate completion and context-aware navigation.

### **Enumerations**
TypeScript offers several tools for crafting clear data definitions, with enumerations and unions standing as popular choices.

**Consider using enums over union types for situations with a fixed set of known values.**

Inevitably you will want to refer to the values of a union type somewhere (perhaps as the argument to a function). You can of course just use a literal which represents a member of that union — but if you have an enum, then all of the values are special, and any time you use a value then anyone can see where that value comes from.

**Don't use numeric enums**

Numeric enums are misleading because it creates a reverse mapping from value to property name, and when using `Object.values` to access member names, it will return the numerical values instead of the member names, potentially causing unexpected behavior.



## **Types**
#### Constrain generic types if necessary
It may not be enough just to have a type or a function take another type — you might have to constrain it if it's not allowed to be anything (e.g. extends Json)

```typescript
// before
function createExampleMiddleware(exampleParam): JsonRpcMiddleware<JsonRpcParams, Json>
// after
function createExampleMiddleware<
  Params extends JsonRpcParams = JsonRpcParmas, 
  Result extends Json = Json
>(exampleParam): JsonRpcMiddleware<Params, Result>
```

#### Use `Partial` to type the accumulating variable in `reduce`
When using reduce in TypeScript, the accumulating variable's type can be difficult to define, especially when dealing with optional properties or incomplete objects. In such cases, using Partial allows you to start with an empty object and gradually add optional properties as needed. Here's an example:

``` typescript
interface User {
  name: string;
  age?: number; // optional property
  email: string;
}

// Partial<User> would be:
type PartialUser = {
  name?: string;
  age?: number;
  email?: string;
}

const users: User[] = [
  { name: "Alice", age: 25, email: "alice@example.com" },
  { name: "Bob", age: 30, email: "bob@example.com" },
];

const oldestUser = users.reduce((acc, user) => {
  // acc starts as Partial<User> (empty object)
  if (user.age && (acc.age === undefined || user.age > acc.age)) {
    acc.age = user.age;
    acc.name = user.name; // add additional properties as needed
  }
  return acc;
}, {} as Partial<User>); // initialize with empty object

console.log(oldestUser); // { name: "Bob", age: 30 }
```


#### Use `Omit` to reduce requirements
`Omit<T, K>` takes two generic types: `T` representing the original object type and `K` representing the property keys you want to remove. It returns a new type that has all the properties of T except for the ones specified in K. Here are some cases to use omit:
- Removing Unnecessary Properties:
  Imagine you have a user interface with optional email and phone number fields. However, your API call only cares about the `username`. You can use Omit to reduce the required properties:
``` typescript
interface User {
  username: string;
  email?: string;
  phoneNumber?: string;
}

// Type for API call payload
type ApiPayload = Omit<User, "email" | "phoneNumber">;

const payload: ApiPayload = { username: "johndoe" };
// Now `payload` only has the `username` property, satisfying the API requirements.
```

- Conditional Omission:
  Sometimes, you might want to remove properties based on a condition. `Omit` can still be helpful:
```typescript
interface CartItem {
  productId: number;
  quantity: number;
  color?: string; // Optional color

// Omit color if quantity is 1
const singleItemPayload = Omit<CartItem, "color" extends string ? "color" : never>;

// Omit color for all items if quantity is always 1
const cartPayload: singleItemPayload[] = [];
```

#### When augmenting an object type with `&`, remember to apply `Omit` to any overlapping property keys.

* `&` does not automatically overwrite overlapping entries like the spread (...) operator does: https://tsplay.dev/NDJ48W
```typescript
type Original = {
  a: string;
  b: number;
  c: boolean;
}
type Augment = {
  c: 'hello'
}
type NoOmit = Original & Augment // never
type UseOmit = Omit<Original, 'c'> & Augment // { a: string; b: number; c: 'hello'; }
```

* One edge case to be especially careful of is if an optional property is being overwritten. Unlike the above case, this will fail silently: https://tsplay.dev/m0OBxN
```typescript
type Original = {
  a: string;
  b: number;
  c?: boolean;
}
type Augment = {
  c?: 'hello'
}
type NoOmit = Original & Augment // { a: string; b: number; c?: undefined; }
```


#### Don't use `any`

* `any` doesn't actually represent "any type"; if you declare a value as `any`, it tells TypeScript to disable typechecking for that value anywhere it appears. This means the compiler won't warn you about potential type mismatches or errors, leading to hidden issues that might surface later in development or production.
* `any` is dangerous because it infects all surrounding and downstream code. For instance, if you have a function that is declared to return `any` which actually returns an object, all properties of that object will be `any`.
* Need to truly represent something as "any value"? Use `unknown` instead.

#### Avoid type assertions, or explain their purpose whenever necessary.
Type assertions inform TypeScript about a variable's actual type, using the `as` syntax or the angle bracket ``<Type>`` syntax.
While type assertions offer flexibility, they can bypass TypeScript's safety net. This means the compiler won't catch potential type mismatches, leaving you responsible for ensuring correctness.


While it's generally recommended to avoid type assertions in TypeScript, here are two common scenarios where they might be necessary:

1. Using reduce:

Imagine you have an array of User objects and want to find the youngest user using reduce. You start with an empty accumulator `({})`, but the final result should be a User object.
```typescript
interface User {
  name: string;
  age: number;
}

const users: User[] = [
  // ... user data
];

const youngestUser = users.reduce((acc, user) => {
  if (!acc.age || user.age < acc.age) {
    acc = user; // Here, `acc` might be undefined initially.
  }
  return acc;
}, {} as User); // Assert `{}` as `User` to satisfy the return type.

console.log(youngestUser); // { name: "...", age: maxAge }
```

2. Iterating over Enum Keys:
   When iterating over enum keys using `Object.keys`, the resulting array is of type `string[]`. However, if you want to access enum member values directly, you might need a type assertion.
```typescript=
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

const directions: Direction[] = [Direction.Up, Direction.Down];

for (const key in directions) {
  const direction = directions[key] as Direction; // Assert to access enum values.
  console.log(direction); // Output: Direction.Up, Direction.Down
}
```

#### Avoid type annotations
Type annotations act like contracts with the compiler, ensuring a variable sticks to a specific data type its entire life.

In rare cases, using type annotations with `reduce` can be beneficial. It helps tp clarify complex accumulator types, enhance readability and ensure the compiler understands its intended use.

If you need to create a data structure with specific values, but ensure that it's declared with a more abstract type. Create a function that takes an argument of the type you want, then use it to build the object.
```typescript
interface Person {
  name: string;
  age: number;
  occupation?: string; // Optional property
}

// Function with type annotations:
function createPerson<T extends { name: string; age: number }>(data: T): Person {
  // Infer types from the data argument
  const { name, age, occupation = "Unknown" } = data;

  // Ensure properties adhere to the Person interface
  return { name, age, occupation };
}

// Usage:
const person1 = createPerson({ name: "Alice", age: 30 }); // { name: "Alice", age: 30, occupation: "Unknown" }
const person2 = createPerson({ name: "Bob", age: 25, occupation: "Developer" }); // { name: "Bob", age: 25, occupation: "Developer" }
```
TypeScript is planning to introduce the `satisfies` keyword, which will provide a more flexible way to declare that a data structure conforms to a specific type without sacrificing readability or maintainability. However, this feature is not yet available.

#### Provide explicit return types for functions/methods
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

#### Prefer `as never` for empty types
never is more accurate for typing an empty container rather than a representation of its fully populated shape.
never is assignable to all types, meaning this practice will rarely cause compiler errors.

🚫
```typescript
export class ExampleClass<S extends BaseState> {
  defaultState: S = {} as S;
  ...
}

obj.reduce(
    (acc, [key, value]) => {
      ...
    },
    {} as Record<keyof typeof obj, Json>,
)
```
```typescript
✅

export class ExampleClass<S extends BaseState> {
  defaultState: S = {} as never;
  ...
}

obj.reduce<Record<keyof typeof obj, Json>>(
    (acc, [key, value]) => {
      ...
    },
    {} as never,
)
```

#### Prefer `as const`
🚫
```typescript
const templateString = `Failed with error message: ${error.message}` // Type 'string'
export const PROVIDER_ERRORS = {
  limitExceeded: () =>
    ({
      jsonrpc: '2.0',
      id: IdGenerator(),
      error: {
        code: -32005,
        message: 'Limit exceeded',
      },
    }),
    ...
}
/** evaluates to:
{
  jsonrpc: number;
  id: number;
  error: {
    code: number;
    message: string;
  };
}
*/
export const SUPPORTED_NETWORK_CHAINIDS = {
  MAINNET: '0x1',
  BSC: '0x38',
  OPTIMISM: '0xa',
  POLYGON: '0x89',
  AVALANCHE: '0xa86a',
  ARBITRUM: '0xa4b1',
  LINEA_MAINNET: '0xe708',
};
// evaluates all values to `string`
```

✅
```typescript
const templateString = `Failed with error message: ${error.message}` as const // Type 'Failed with error message: ${typeof error.message}'
export const PROVIDER_ERRORS = {
  limitExceeded: () =>
    ({
      jsonrpc: '2.0',
      id: IdGenerator(),
      error: {
        code: -32005,
        message: 'Limit exceeded',
      },
    } as const),
  ...
}
/** evaluates to:
{
  jsonrpc: '2.0';
  id: ReturnType<IdGenerator>;
  error: {
    code: -32005;
    message: 'Limit exceeded';
  };
}
*/
export const SUPPORTED_NETWORK_CHAINIDS = {
  MAINNET: '0x1',
  BSC: '0x38',
  OPTIMISM: '0xa',
  POLYGON: '0x89',
  AVALANCHE: '0xa86a',
  ARBITRUM: '0xa4b1',
  LINEA_MAINNET: '0xe708',
} as const;
```

Exceptions:
🚫
```typescript
// This isn't necessary because the string literal is already considered a const by the compiler.
const walletName = 'metamask' as const;
this.messagingSystem.registerActionHandler(
      `${name}:call` as const,
      ...
);
```

✅
```typescript
const walletName = 'metamask';
this.messagingSystem.registerActionHandler(
      `${name}:call`,
      ...
);
```


## **Language features**
#### Prefer the nullish coalescing operator `??`
* `??` guarantees that the left-hand operand is nullish but not falsy, which is both safer and reduces cognitive overhead for reviewers.
* Using `||` leads reviewers to assume that ?? was intentionally avoided, forcing them to consider cases where the left-hand operand is falsy. This should never happen if the left-hand operand is guaranteed to be nullish and not falsy.

#### Prefer negation (`!`) over strict equality checks for `undefined`, but not for `null`.
* Unlike with vanilla JavaScript, in TypeScript, we can rely on type inference and narrowing to correctly handle `null` checks without using strict equals(`===`).
* This does not apply if the `null` check needs to exclude falsy values.
* Because our codebase usually uses `undefined` for empty values, if a `null` is used, it should be assumed to be an intentional choice to represent an expected exception case or a specific class of empty value.

#### `Object.entries`
* `Object.keys` and `Object.entries` both type the object keys (index signature) as `string` instead of `keyof typeof` obj. An annotation or assertion is necessary to accurately narrow the index signature.
* Strong typing for the `entries` operation using generic arguments is unavailable, as the generic parameter for `Object.entries` only allows typing the object's values.
  🚫
```typescript
Object.entries<Json>(state).forEach(([key, value]) => {
  ...
}
```
✅
```typescript
(Object.keys(state) as (keyof typeof state)[]).forEach((key) => {
  ...
}

Object.entries(state).forEach(([key, value]): [keyof typeof state, Json]) => {
  ...
}
```

#### `Array.prototype.reduce`
* Use the generic argument to type the fully populated accumulator.
* Type the initial value of the accumulator object as `never`.

🚫
```typescript
Object.entries(state).reduce(
    (acc, [key, value]: [keyof typeof state, Json]) => {
      ...
    },
    {} as Record<keyof typeof state, Json>,
)
Object.entries(state).reduce<Partial<Record<keyof typeof state, Json>>>(
    (acc, [key, value]: [keyof typeof state, Json]) => {
      ...
    },
    {},
) as Record<keyof typeof state, Json>
```
✅
```typescript
Object.entries(state).reduce<Record<keyof typeof state, Json>>(
    (acc, [key, value]: [keyof typeof state, Json]) => {
      ...
    },
    {} as never,
)
```

#### Function supertype
🚫
```typescript
type AnyFunction = (...args: any[]) => any;

// Safe to use only if function params are guaranteed to be fully variadic with no fixed parameters.
type NeverFunction = (...args: never[]) => unknown;
```
✅
```typescript
type FunctionSupertype = ((...args: never) => unknown) | ((...args: never[]) => unknown)
```
1. `(...args: never[]) => unknown`

The return type of this supertype function needs to be `unknown` (universal supertype), and the parameters need to be `never` (universal subtype).

This is because function parameters are contravariant, meaning `(x: SuperType) => T` is a subtype of `(x: SubType) => T`.

* There's an example of this in the [TypeScript docs](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types).
* And here's a more detailed [reddit comment](https://www.reddit.com/r/typescript/comments/muyl55/comment/gv9ndij/?utm_source=share&utm_medium=web2x&context=3) on this topic.

2. `(...args: never) => unknown`

In general, array types of arbitrary length are not assignable to tuple types that have fixed types designated to some or all of its index positions (the array is wider than the tuple since it has less constraints).

This means that there's a whole subcategory of functions that `(...args: never[]) => unknown` doesn't cover: functions that have a finite number of parameters that are strictly ordered and typed, while also accepting an arbitrary number of "rest" parameters. The spread argument list of such a function evaluates as a variadic tuple type with a minimum fixed length, instead of a freely expandable (or contractable) array type.

`(...args: never[]) => unknown` covers functions for which all of the params can be typed together as a group e.g. `(...args: (string | number)[]) => T`, or it has a fixed number of params e.g. `(a: string, b: number) => T`. But to cover cases like `(a: string, b: number, ...rest: unknown[]) => T`, we need another top type with an even narrower type for the params: `(...args: never) => unknown`.


#### Avoid implicit, hard-coded generic parameters that are inaccessible from the outermost user-facing type.
* These are also called "return type-only generics" and are discouraged by the [Google TypeScript style guide](https://google.github.io/styleguide/tsguide.html#return-type-only-generics).
* Implicit generic parameters make it difficult to debug typing issues, and also makes the type brittle against future updates.
* Exception: If the intention is to encapsulate certain generic parameters as "private" fields, wrapping the type in another type that only exposes "public" generic parameters is a good approach.


## Further reading
* [Official Typescript documents of Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
