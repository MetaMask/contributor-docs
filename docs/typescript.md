# Typescript Guidelines

**_Purpose_**: Establish best practices for writing robust, maintainable, and efficient TypeScript code.

**_Audience_**: Internal and external developers using TypeScript to contribute to MetaMask.

### Implements

#### `implements` keyword

The `implements` keyword enables us to define and enforce interfaces, i.e. strict contracts consisting of expected object and class properties and abstract method signatures.
Writing an interface to establish the specifications of a class that external code can interact while without being aware of internal implementation details is encouraged as sound OOP development practice.
Here's an abbreviated example from `@metamask/polling-controller` of an interface being used to define one of our most important constructs.

```typescript
export type IPollingController = {
...
}

export function AbstractPollingControllerBaseMixin<TBase extends Constructor>(
    Base: TBase,
) {
    abstract class AbstractPollingControllerBase
        extends Base
        implements IPollingController
    { ... }
    return AbstractPollingControllerBase
}
```

The concept of the interface as discussed in this section is not to be confused with interface syntax as opposed to type alias syntax. Note that in the above example, the `IPollingController` interface is defined as a type alias, not using the `interface` keyword.

#### Always prefer type aliases over `interface` keyword

We enforce consistent and exclusive usage of type aliases over the `interface` keyword to declare types for several reasons:

- The capabilities of type aliases is a strict superset of those of interfaces.
  - Crucially, `extends`, `implements` are also supported by type aliases.
  - Declaration merging is the only exception, but we have no use case for this feature that cannot be substituted by using type intersections.
- Unlike interfaces, type aliases extend `Record` and have an index signature of `string` by default, which makes them compatible with our Json-serializable types (most notably `Record<string, Json>`).
- Interfaces do not support multiple inheritance, while type aliases can be freely merged using the intersection (`&`) operator.

### Enums

TypeScript offers several tools for crafting clear data definitions, with enumerations and unions standing as popular choices.

#### Consider using enums over union types for situations with a fixed set of known values.

Inevitably you will want to refer to the values of a union type somewhere (perhaps as the argument to a function). You can of course just use a literal which represents a member of that union — but if you have an enum, then all of the values are special, and any time you use a value then anyone can see where that value comes from.

🚫

```typescript
type UserRole = 'admin' | 'editor' | 'subscriber';
```

✅

```typescript
enum AccountType {
  Admin = 'admin',
  User = 'user',
  Guest = 'guest',
}
```

#### Don't use numeric enums

Numeric enums are misleading because it creates a reverse mapping from value to property name, and when using `Object.values` to access member names, it will return the numerical values instead of the member names, potentially causing unexpected behavior.
🚫

```typescript
enum Direction {
  Up = 0,
  Down = 1,
  Left = 2,
  Right = 3,
}

const directions = Object.values(Direction); // [0, 1, 2, 3]
```

✅

```typescript
enum Direction {
  Up = 'Up',
  Down = 'Down',
  Left = 'Left',
  Right = 'Right',
}

const directions = Object.values(Direction); // ["Up", "Down", "Left", "Right"]
```

## Types

#### Constrain generic types if necessary

It may not be enough just to have a type or a function take another type — you might have to constrain it if it's not allowed to be anything (e.g. extends Json)

```typescript
// before
function createExampleMiddleware<Params, Result>(exampleParam);
// after
function createExampleMiddleware<
  Params extends JsonRpcParams,
  Result extends Json,
>(exampleParam);
```

#### Use `Omit` to reduce requirements

`Omit<T, K>` takes two generic types: `T` representing the original object type and `K` representing the property keys you want to remove. It returns a new type that has all the properties of T except for the ones specified in K. Here are some cases to use omit:

- Removing Unnecessary Properties:
  Imagine you have a user interface with optional email and phone number fields. However, your API call only cares about the `username`. You can use Omit to reduce the required properties:

```typescript
interface User {
  username: string;
  email?: string;
  phoneNumber?: string;
}

// Type for API call payload
type ApiPayload = Omit<User, 'email' | 'phoneNumber'>;

const payload: ApiPayload = { username: 'johndoe' };
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

#### Avoid type assertions, or explain their purpose whenever necessary.

Type assertions inform TypeScript about a variable's actual type, using the `as` syntax.
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

```typescript
enum Direction {
  Up = 'up',
  Down = 'down',
  Left = 'left',
  Right = 'right',
}
const directions: Direction[] = [Direction.Up, Direction.Down];
for (const key of Object.keys(directions)) {
  const direction = directions[key as keyof typeof directions];
  console.log(direction); // Output: Direction.Up, Direction.Down }
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
function createPerson<T extends { name: string; age: number }>(
  data: T,
): Person {
  // Infer types from the data argument
  const { name, age, occupation = 'Unknown' } = data;

  // Ensure properties adhere to the Person interface
  return { name, age, occupation };
}

// Usage:
const person1 = createPerson({ name: 'Alice', age: 30 }); // { name: "Alice", age: 30, occupation: "Unknown" }
const person2 = createPerson({ name: 'Bob', age: 25, occupation: 'Developer' }); // { name: "Bob", age: 25, occupation: "Developer" }
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

## Further reading

- [Official Typescript documents of Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
