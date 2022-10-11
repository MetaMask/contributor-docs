# JavaScript

## Use the latest version of our ESLint config

If you're working on a project, there's a good chance that you're using our shared ESLint rules, which codifies decisions we've made around code style and can be used to align that project with other projects in our ecosystem. Make sure the project is up to date with the various packages within the [`eslint-config`](https://github.com/MetaMask/eslint-config) monorepo.

## Use a verb phrase for function/method names

When naming a function or method, describe the action, not the outcome.

🚫

``` javascript
function formattedChangelog() {
  // ...
}
```

✅

``` javascript
function formatChangelog() {
  // ...
}
```

For a function or method that returns a boolean, prefix the name with a descriptive verb so that readers don't confuse it with a boolean variable.

🚫

``` javascript
class NetworkController {
  isEIP1559Compatible() {
    // ...
  }
}
```

✅

``` javascript
class NetworkController {
  determineEIP1559Compatibility() {
    // ...
  }
}
```

### Read more

- [Google's JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html#naming-method-names)
- [.NET Fundamentals](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members#names-of-methods)
- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html#choose-good-names)
- [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/#strive-for-fluent-usage)

## Use a verb phrase for boolean variable names that do not involve secondary objects

Sometimes a variable that is intended to hold a boolean value does not have an explicit subject, but represents the context where the variable is defined (e.g., a class or an entire file). When naming such a variable, use a statement that describes the state of the subject, minus the name of the subject itself. Usually this means prefixing the name with a form of "to be" or "to have" (e.g. `is*`, `has*`), but you may find it more readable to use past or future tense and/or a modal verb such as `should`.

🚫

``` javascript
const removed = false;
```

✅

``` javascript
// Any of these would do
const isRemoved = false;
const wasRemoved = false;
const hasBeenRemoved = false;
const shouldBeRemoved = false;
```

If the name represents a negative statement, reword it into a positive statement and inverting the value assigned to the value.

🚫

``` javascript
const notEnoughGas = false;
```

✅

``` javascript
const hasEnoughGas = true;
```

Take special note of variables which are created via React's `useState` hook.

🚫

``` javascript
const [removed, setRemoved] = useState(false);
```

✅

``` javascript
const [isRemoved, setIsRemoved] = useState(false);
```

## 💡 Place names of secondary concepts first in boolean variable names

Sometimes a boolean variable describes a subject that's different than the context where the variable is defined:

``` javascript
const ownedAccount = Boolean(ownedAccountName);
```

In naming such a variable, there are a couple of different routes you could take.

One way is to always place the verb at the beginning:

``` javascript
const isRecipientOwnedAccount = Boolean(ownedAccountName);
```

However, this naming strategy creates a point of friction for objects, arrays, or React components, where it may be desirable to sort identifiers alphabetically. In that case you could end up with something like:

``` jsx
<SenderToRecipient
  isRecipientOwnedAccount={isRecipientOwnedAccount}
  recipientName={toName}
  recipientNickname={toNickname}
  senderName={fromName}
  senderAddress={fromAddress}
/>
```

It is potentially more "greppable" if properties that concern the same concept are kept together instead of separate. To address this, you may wish to place the subject of the variable name at the beginning:

``` javascript
const recipientIsOwnedAccount = Boolean(ownedAccountName);
```

This would result in:

``` jsx
<SenderToRecipient
  recipientIsOwnedAccount={isRecipientOwnedAccount}
  recipientName={toName}
  recipientNickname={toNickname}
  senderName={fromName}
  senderAddress={fromAddress}
/>
```

## Use `async`/`await` syntax over `.then`/`.catch`

Asynchronous code written using `async`/`await` syntax looks less complex and more straightforward than code written using `.then`/`.catch`. Additionally, using `async`/`await` leads to [better stack traces in Node and Chromium](https://mathiasbynens.be/notes/async-stack-traces), both of which use the V8 JavaScript engine, because when an asynchronous operation is `await`ed, the engine will remember the function where the `await` occurred, which means that it can place that function on the stack trace (otherwise, using `.then`/`.catch`, it would get lost).

🚫

``` javascript
function makeRequest() {
  return fetch("https://google.com")
    .then((response) => {
      return response.json().then((json) => {
        return json['page_views'];
      })
    })
}
```

✅

``` javascript
async function makeRequest() {
  const response = await fetch("https://google.com");
  const json = await response.json();
  return json['page_views'];
}
```

## `await` promises before `return`ing them

Using `return await` rather than just `return` for an asynchronous operation leads to a more complete stack trace in Node and in Chromium browsers, both of which use the V8 JavaScript engine, because when an asynchronous operation is `await`ed, the engine will remember the function where the `await` occurred, which means that it can place that function on the stack trace (otherwise it would get lost).

🚫

``` javascript
async function makeRequest() {
  const response = await fetch('https://some/url');
  return response.json();
}
```

✅

``` javascript
async function makeRequest() {
  const response = await fetch('https://some/url');
  return await response.json();
}
```

### Read more

* <https://v8.dev/blog/fast-async#improved-developer-experience>
* <https://docs.google.com/document/d/13Sy_kBIJGP0XT34V1CV3nkWya4TwYx9L3Yv45LdGB6Q/edit>

## Use Jest's mock functions instead of Sinon

Jest incorporates most of the features of Sinon with a slimmer API:

* `jest.fn()` can be used in place of `sinon.stub()`.
* `jest.spyOn(object, method) can be used in place of `sinon.spy(object, method)`.
* `jest.spyOn(object, method)` can be used in place of `sinon.stub(object, method)` (with the caveat that the method being spied upon will still be called by default).
* `jest.useFakeTimers()` can be used in place of `sinon.useFakeTimers()` (though note that Jest's "clock" object has a different API than Sinon's and offers fewer options for advancing time).

## Don't test private code

If you encounter a function or method whose name starts with `\_` or which has been tagged with the `@private` JSDoc tag or `private` keyword in TypeScript, you do not need to test this code. These markers are used to indicate that the code in question should not be used outside the file. As a result, any tests written for this code would need to have knowledge that no other part of the system would have. This is not to say that private code should not be tested at all, but rather that it should be tested via public code that makes use of the private code.
