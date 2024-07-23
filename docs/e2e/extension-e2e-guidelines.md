# MetaMask Extension E2E Test Guidelines

### TL;DR

- Manage test size and execution time
- Craft reliable element locators using data-testid
- Avoid using arbitrary delays, leverage wait commands
- Write repeatable tests that produce the same result every time
- Explicitly state assertions
- Enhance test stability with request mocking
- Utilize [Page Object Model (POM)](#page-object-model-pom) for streamlined test automation

### Key

✅ Recommend⚠️ Use with caution, see notes ❌ Avoid

## Managing Test Size and Execution Time

To maintain efficient and effective E2E testing within MetaMask Extension repository, it's crucial to address the execution time of our spec files. Here are key considerations:

- Limiting Tests per Spec File :
  It's advisable to avoid overcrowding spec files with too many tests. A high number of tests within a single spec file can lead to longer execution times and complicate the process of identifying and isolating failures.

- Optimizing Test Execution Time :
  Aiming for each spec file's runtime to be under approximately 2 minutes ensures tests are executed swiftly, facilitating quicker feedback loops. This time frame helps in maintaining a balance between comprehensive testing and efficient use of testing resources.

- Rationale
  The structure of spec files plays a pivotal role in our testing strategy, especially considering the functionality provided by CircleCI. CircleCI treats each spec file as the smallest unit for splitting and retrying tests. Consequently, if a spec file contains 10 tests and one fails, all 10 tests must be retried, impacting the efficiency of our testing process. This approach also applies to the "Rerun failed tests" feature.

While it's theoretically possible to split tests by individual cases on CircleCI, such an approach requires significant effort and is not currently implemented. By optimizing the number of tests per spec file and their execution time, we can enhance the overall efficiency and effectiveness of our E2E testing.

## Element locators

Crafting resilient locators is crucial for reliable tests. It’s important to write selectors that are resilient to changes in the extension's UI. Tests become less prone to issues as the extension is updated, for example, during UI redesigns. This results in less effort required to maintain and update the tests, improving the stability of the tests and reducing the associated maintenance costs. Element locators should be independent of CSS or JS so that they do not break on the slightest UI change. Another consideration is whether a test should fail if the content of an element changes. It's worth noting that in the testing framework, element locators extend beyond simple CSS selector strings. They can also be objects or methods that return strings or objects, providing flexible and resilient ways to identify elements.

### Guidelines

✅ Resilient to changes

```javascript
await driver.clickElement('[data-testid="page-container-footer-next"]');
```

✅ Ok to check the expected text as it is part of the user's experience

```javascript
await driver.clickElement({ text: 'Next', tag: 'button' });
```

⚠️ Coupled with styling: changes to how buttons are styled will cause this locator to fail

```javascript
await driver.clickElement('.btn-primary');
```

❌ Chaining direct descendants and using indexes: changes to one element position in the chain will cause this locator to fail

```javascript
await driver.clickElement('footer > button:nth-of-type(2)');
```

❌ Xpath locators tend to be slower and more difficult to understand than CSS

```javascript
await driver.clickElement('//*[@data-testid="page-container-footer-next"]')';
```

Here are some examples how to improve selectors following above guidelines:

```javascript
// current element locator:
.qr-code__wrapper
// recommanded element locator: replace CSS locator with a data-testid
'[data-testid="account-details-qr-code"]'

// current element locator:
'//div[contains(@class, 'home-notification__text') and contains(text(), 'Backup your Secret Recovery Phrase to keep your wallet and funds secure')]'
// recommanded element locator: replace XPATH with a query
'{ text: Backup your Secret Recovery Phrase to keep your wallet and funds secure, tag: div }'
```

## Wait for commands

Some steps in an end-to-end test require a condition to be met before running. It is tempting to use "sleep" calls, which cause the test to pause execution for a fixed period of time. This approach is inefficient: sleeping for too long may unnecessarily lengthen the execution and sleeping for too short may lead to an intermittently failing test. Instead, leverage "wait-for" commands, which still pause but ensure the tests continue as soon as the desired state is reached, making the tests more reliable and performant.

💡 When is it necessary to wait?

- Before Locating the Element:
  Ensure that the page or relevant components have fully loaded. Use explicit waits to wait for certain conditions, like the visibility of an element or the presence of an element in the DOM.

- CSS Selector and Element Existence:
  Ensure that the CSS selector is correct and that the targeted element actually exists in the DOM at the time of the search. It's possible the element is dynamically loaded or changed due to a recent update in the application.

- Context Switching:
  Consider switching the context to that iframe or modal before attempting to locate the element.

- Recent Changes:
  If the issue started occurring recently, review any changes made to the application that could affect the visibility or availability of the element.

- Timeout Period:
  If the default timeout is too short for the page or element to load, consider increasing the timeout period. This is especially useful for pages that take longer to load due to network latency or heavy JavaScript use.

### Guidelines

✅ Explicit wait: wait until a condition occurs.

```javascript
await driver.wait(async () => {
  info = await getBackupJson();
  return info !== null;
}, 10000);
```

✅ Explicit wait: wait until a condition occurs.

```javascript
await driver.wait(async () => {
  const isPending = await mockedEndpoint.isPending();
  return isPending === false;
}, 3000);
```

✅ Explicit wait for element to load

```javascript
await driver.waitForSelector('.import-srp__actions');
```

✅ Explicit wait for element no present especially when navigating pages

```javascript
await driver.waitForElementNotPresent('.loading-overlay__spinner');
```

❌ SetTimeout: the hard-coded wait may be longer than needed, resulting in slower test execution.

```javascript
await driver.delay(timeInMillis);
```

❌ SetTimeout: The hard-coded wait may be longer than needed, resulting in slower test execution.

```javascript
await new Promise((resolve) => setTimeout(resolve, timeInMillis));
```

### Repeatable tests

Avoiding conditional statements in the end-to-end tests promotes simpler and more maintainable tests. A clear linear flow helps with understanding especially when it comes to identifying and fixing problems in failed tests. Changes in the extension's behaviour can easily break tests that rely on specific conditions to be met. It’s important to write repeatable tests that produce the same result every time they are run regardless of the environment. Conditional statements add variability to the tests as well as increase the test's complexity, making it harder to control the outcome. Changes to the test will need to be made in multiple places as the test logic is spread across branches.

### Guidelines

❌ Avoid following statements as they introduce complexity and variability into the tests:

```javascript
- If statement
- Ternary operator
- Switch
- Short circuiting

//example code:
if (type !== signatureRequestType.signTypedData)
```

✅ Proposed solution: Remove conditional statements in test, structure the tests in a way that avoids branching logic.

## Assertions

Assertions should verify expected behaviour and outcomes during test execution. When tests are responsible for assertions rather than being hidden in helper functions, it promotes test readability as expected results are explicitly stated. This makes it easier for engineers to quickly understand the intent of the test as well as the cause of failures.

### Guidelines

✅ Explicit assertion: clear and concise error message

```javascript
const isExpectedBoxContentPresentAndVisible =
  await driver.isElementPresentAndVisible({
    css: '[data-testid="mm-banner-alert-notification-text"]',
    text: options.text,
  });
assert.equal(
  isExpectedBoxContentPresentAndVisible,
  true,
  'Invalid box text content',
);

// Error message:
// AssertionError [ERR_ASSERTION]: Expected value to be true:
// + actual - expected
// -'Invalid box text content'
```

❌ Race condition: it is possible that the completedTx is found before component rendering is complete, and then completedTx.getText() can return values that don't reflect the final state of the data, and don't reflect what the user will actually see.

```javascript
const completedTx = await driver.findElement(
  '[data-testid="transaction-list-item"]',
);
assert.equal(await completedTx.getText(), 'Send TST');

// Error message:
// AssertionError [ERR_ASSERTION]: Expected values to be strictly equal:
// + actual - expected
// + 'Send Token'
// - 'Send TST'
```

✅ Assertion which provide clear and concise error messages makes it easier to diagnose and fix issues in tests:

```javascript
assert.equal(await qrCode.isDisplayed(), true, ‘The QR code should be displayed’);
// error: The QR code should be displayed + expected - actual
//		-false
//		+ true
```

❌ Assertion with unclear error message:

```javascript
assert.equal(await qrCode.isDisplayed(), true);
// AssertionError [ERR_ASSERTION]: Expected values to be strictly equal: false !== true
```

## Enhancing test stability with request mocking

Intercepting network requests and substituting responses with predefined mocks significantly improves the speed and stability of the end-to-end tests by eliminating reliance on external services. This approach not only provides greater control over APIs, allowing a wide range of scenarios including network errors to be tested, but also helps to verify the extension's behaviour under adverse network conditions.

It's important to note that third-party websites and applications, which are beyond the control of the MetaMask engineering team, may not always behave consistently or be available. Relying on these external services in tests increases the chance of failures due to unknown factors such as content changes, service issues, or unstable network connections. Therefore, you should aim to minimise such dependencies and be prepared for intermittent access issues.

### Guidelines for intercepting network requests

Control the network by intercepting requests or waiting for requests to be responded to.

- This could be a scam

```javascript
// File: test/e2e/tests/security-provider.spec.js
return {
  statusCode: 200,
  json: {
    flagAsDangerous: 1,
  },
};
```

- Request may not be safe: change the response body to mock a different scenario

```javascript
// File: test/e2e/tests/security-provider.spec.js
return {
  statusCode: 200,
  json: {
    flagAsDangerous: 2,
  },
};
```

- Request not verified: change the response code to simulate different response classes

```javascript
// File: test/e2e/tests/security-provider.spec.js
return {
  statusCode: 500,
  json: {},
};
```

- Waits for a request to be responded to.

```javascript
// File: test/e2e/tests/metrics.spec.js
await driver.wait(async () => {
  const isPending = await mockedEndpoint.isPending();
  return isPending === false;
}, 10000);
```

### Guidelines for limiting third-party calls

Reduce reliance on external services and using techniques like response mocking and local simulations in order to control unpredictability and enhance test reliability.

✅ Full control over the dapp by using versions. No internet connection required.

```javascript
Type: E2E Test Dapp
Url: http://localhost:8080
```

⚠️ Full control over changes or updates to the dapp by using versions. Requires an internet connection

```javascript
Type: Test Snaps
Url: https://metamask.github.io/test-snaps/5.1.0/
```

⚠️ Partial control over changes or updates to the dapp. Requires an internet connection.

```javascript
Type: E2E Test Dapp
Url: https://metamask.github.io/test-dapp/
```

❌ No control over changes or updates to the dapp. Requires an internet connection.

```javascript
Type: Dapp
Url: https://app.uniswap.org/#/swap
```

❌ No control for external dependency.

```javascript
File: test/e2e/tests/import-flow.spec.js
Test Name: Connects to a Hardware wallet for Trezor
```

✅ Proposed solution: The Trezor import flow involves opening the Trezor website, then the user takes additional steps on that website to connect the device. A fake version of this website could be created for testing, and the test build could be updated to use this fake version. It's also worth investigating a phishing detection solution, such as replacing Github.com with an empty page.

## Page Object Model (POM)

POM, or Page Object Model, is a design pattern commonly utilized for writing automated end-to-end tests. A page object is a instance of a class that acts as an interface for the page of the application under test.
This design pattern is being adopted due to its numerous benefits, including improved test maintenance, increased code reusability, and enhanced readability of test scripts. By abstracting the UI details into page objects, changes to the application's UI require updates only in the page objects, significantly reducing the effort needed to maintain tests as the application evolves. The adoption of POM in our e2e testing strategy aims to leverage these advantages to create a more robust, maintainable, and efficient testing framework.

### Composition of a Page Class

Each page class is composed of:

- **Locators**: HTML elements.
- **Action Methods**: Methods for interacting with the elements.
- **Check Methods**: Methods that assert the status of elements.

A test creates page objects and interacts with web elements by calling methods of those page objects.

### Best Practices

- A page object should represent meaningful elements of a page and not necessarily a complete page. It can represent a component of a page, like a navbar. See example [here.](https://github.com/MetaMask/contributor-docs/examples/extension-e2e-page-object-model/homepage.ts#L6)
- All locators should be kept in the page object file. Carefully define locators for elements in page objects, opting for robust locators since they will be extensively used across various locations. For guidance on crafting these resilient locators, refer to the [element locators section](#element-locators), which outlines the recommended strategies for their creation and usage.
- Page objects should remain independent and not invoke other page objects to prevent circular references, ensuring they are typically isolated from each other. For handling complex workflows that require interaction across multiple pages, _processes_ should be implemented. Check out the [implementation here.](https://github.com/MetaMask/contributor-docs/examples/extension-e2e-page-object-model/login.process.ts#L14) This approach enables the incorporation of all relevant page objects to support specific flows, such as login, sending a transaction, or creating a swap. A dedicated `processes` folder is used to organize and manage these complex workflows.
- The tests should only call page object methods or processes, they shouldn't interact directly with page elements. [See example here.](https://github.com/MetaMask/contributor-docs/examples/extension-e2e-page-object-model/simple-send.spec.ts#L9)
- Page object methods should include detailed logs and detailed error messages in all check methods to aid in debugging tests.
- Place assertions inside of `check_` methods, and call `check_` methods inside of tests instead of making assertions directly. Along with enhanced logging, this ensures that `check_` methods are reusable across different tests. [See example here.](https://github.com/MetaMask/contributor-docs/examples/extension-e2e-page-object-model/homepage.ts#L117)
- Page objects and tests should be written in TypeScript.
- Follow the naming conventions outlined below for page objects, locators, and methods.

### Naming Convention

#### Page Objects

- Classes start with a capital letter following the word “page”, e.g., `LoginPage`.
- File names are a kebab-case version of the class name, e.g. `LoginPage` would be contained in a file called `login-page.ts`.

#### Locators

- Locators should be suffixed by the type of the element they represent, e.g., `submitButton`, `passwordInput`, etc.

#### Action Methods

- Follow the camelCase standard, with names that clearly indicate an action and are self-explanatory, e.g., `confirmTx()`.

#### Check Methods

- Using `check_` followed by camelCase for all check methods effectively distinguishes them from action methods, e.g., `check_expectedBalanceIsDisplayed()`. These methods perform the same function as `assert()` statements in the current test body. This naming convention ensures that check methods are as prominent as `assert()` statements in the existing code. This approach proves especially advantageous for long test bodies.
