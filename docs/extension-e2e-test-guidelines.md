# MetaMask Extension E2E Test Guidelines

## General

The primary goal of writing end-to-end (E2E) tests at MetaMask is to ensure that all of the user flows within the product function as designed. Therefore, test coverage is a crucial metric for evaluating and improving quality. The higher the coverage is, the more confidence this creates among engineers in the product, and the more effectively bugs can be identified and eliminated.

The process of creating an E2E test involves several steps: identification, development, testing, and maintenance. Initially, it is important to identify the right aspects of the product to test. The next step is to develop the tests, adhering to the quality guidelines outlined in this document. This includes creating clear test cases, using appropriate assertions, and structuring the tests for easy maintenance. Finally, it's crucial to ensure that tests are robust and reliable. This means the test should consistently produce the same results and be resilient to minor system changes. By following these steps, effective E2E tests can be created that contribute to delivering a high-quality product.

This document serves as a guideline to approach writing E2E tests so that they provide fast feedback, they become less fragile over time, they are painless to debug, and they are scalable and easy to maintain. The remainder of this document will attempt to explain the suggested approach with relevant examples to help better understand the future direction for tests.

These guidelines aren't meant to be a strict set of rules, they should be thought of as an approach to writing robust end-to-end tests. Use them where they make sense and adapt them as necessary to complement your tests.

### TL;DR

- Write test names that explain the behaviour of the test
- Organization of test files
- Craft reliable element locators using data-testid
- Avoid using arbitrary delays, leverage wait commands
- Only test things that you can control
- Explicitly state assertions
- Control the extension state without going through the UI
- Avoid relying on the completion of another test in order for a test to be successful
- Write repeatable tests that produce the same result every time
- Keep testing modules up to date

### Key

✅ Recommend⚠️ Use with caution, see notes ❌ Avoid

## Test names

The test name should communicate the purpose and behaviour of the test. A clear test name serves as a concise summary of what the test aims to verify, making it easier for everyone to understand and maintain the tests. Clear test names improve the readability of the test and help when it comes to debugging a failed test. You should be able to figure out the purpose of the test without going through the complete implementation. If the test is failing, it should be easy to figure out which functionality is broken from the test name.

### Guidelines

✅ Recommended readable test names

```javascript
- adds Bob to the address book
- send 1 TST to Bob
```

⚠️ Test names remains approachable, recommend to avoid using the prefix meaningless 'should'

```javascript
- should add Bob to the address book
- should send 1 TST to Bob
```

❌ Test name should be completely avoided: meaningless `should` prefix, the `and` decreases the readability of the test,making it harder to understand what the test is doing as well as diagnose and fix issues.

```javascript
- should add Bob to the address book and send 1 TST to Bob
```

✅ Recommended readable test names

```javascript
- removes an account imported with a private key
- impossible to remove an account generated from the SRP imported in onboarding

- the UI environment is locked down
- the background environment is locked
```

❌ Test name should be completely avoided

```javascript
// in file: test/e2e/tests/add-account.spec.js
- should be possible to remove an account imported with a private key, but should not be possible to remove an account generated from the SRP imported in onboarding

// in file: test/e2e/tests/lockdown.spec.js
- the UI and background environments are locked down
```

## Organization of test files

It's essential to organise your test files. As test suites get bigger, a well-structured organisation makes it easier to search for tests as well as identify logical groups of tests that may be impacted by changes to an area of the extension. When tests are organised based on related features or functionality it becomes easier to identify common helper functions that can be shared across the tests, reducing duplication.

### Proposal

Organise tests into folders based on scenarios and features. This means that each type of scenario has its own folder, and each feature team owns one or more folders.

- This approach provides ownership of E2E testing at the feature team level. Each feature team is aware of the features they own, making it easier for them to understand what tests we currently have and what's missing.
- In the future, we aim to have specific feature teams manage the necessary test cases for their features, rather than having the core teams keep track of each one.
- Using this organizational strategy eliminates the need to decide at a low level where to place a test, as it is straightforward based on the feature or scenario.
- This approach aligns with the testing strategy for the mobile app. The mobile team prefers to have feature team tests under the same folder. This reduces friction when switching context.

Example for organization of test files by features and scenarios:

```javascript
// current test path:
test/e2e/tests/nft/import-erc1155.spec.js
// recommanded test path: (consolidate all import tests for different tokens into a single repository)
test/e2e/tests/tokens/import/import-erc1155.spec.js

// current test path:
test/e2e/tests/clear-activity.spec.js
// recommanded test path:
test/e2e/tests/settings/clear-activity.spec.js

// current test path:
test/e2e/tests/ppom-blockaid-alert-erc20-approval.spec.js
// recommanded test path:
test/e2e/tests/ppom/ppom-blockaid-alert-erc20-approval.spec.js
```

## Element locators

Crafting resilient locators is crucial for reliable tests. It’s important to write selectors resilient to changes in the extension's UI. Tests become less prone to issues as the extension is updated e.g. UI redesigns. As a result, less effort is required to maintain and update the tests, improving the stability of the tests and reducing the associated maintenance costs. Element locators should be independent of CSS or JS so that they do not break on the slightest UI change. Another thing to consider is whether we would want our test to fail if the content of an element changes.

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

💡 When do we need to wait?

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

### Proposal

Provide clear and concise error messages in test assertions making it easier to diagnose and fix issues in the tests.

❌ Current assertion and current error message:

```javascript
assert.equal(await qrCode.isDisplayed(), true);
// AssertionError [ERR_ASSERTION]: Expected values to be strictly equal: false !== true
```

✅ Proposed assertion and error message if we adopt the proposal:

```javascript
assert.equal(await qrCode.isDisplayed(), true, ‘The QR code should be displayed’);
// error: The QR code should be displayed + expected - actual
//		-false
//		+ true
```

## Controlling state

To achieve test atomicity and ensure our E2E tests are stable and reliable, we need to control the state of the extension programmatically, rather than relying on the application UI. Setting the state programmatically eliminates unnecessary UI interactions, decreasing the amount of possible breaking points in a test. It improves test stability by minimising issues caused by timing synchronisation or inconsistencies in the UI, reducing the test execution time and allowing the test to provide fast and focused feedback.

### Guidelines

✅ Use fixture to remove multiple redundant steps: Add Contact, Open test dapp, Connect to test dapp, Deploy TST, Add TST to wallet

```javascript
new FixtureBuilder()
  .withAddressBookControllerContactBob()
  .withTokensControllerERC20()
  .build();
```

Send TST test steps after setting this fixture:

```javascript
Login
Send TST
Assertion
```

⚠️ Use fixture to remove single redundant steps: Add Contact

```javascript
new FixtureBuilder().withAddressBookControllerContactBob().build();
```

Send TST test steps after setting this fixture:

```javascript
Login
Open test dapp
Connect to test dapp
Deploy TST
Add TST to wallet
Send TST
Assertion
```

❌ Use the UI to build state

```javascript
new FixtureBuilder().build();
```

Send TST test steps after setting this fixture:

```javascript
Login
Add Contact
Open test dapp
Connect to test dapp
Deploy TST
Add TST to wallet
Send TST
Assertion
```

### Proposal

Identify opportunities to use the FixtureBuilder to create the state, instead of navigating through the UI. Here are some examples:

1.

```javascript
// test file: test/e2e/tests/multiple-transactions.spec.js
scenario: creates multiple queued transactions and then confirms.
solution: Ensure tests are different from the tests that line in test/e2e/tests/navigate-transactions.spec.js
```

2.

```javascript
// test file: test/e2e/tests/multiple-transactions.spec.js
scenario: creates multiple queued transactions, then rejects.
solution: create multiple transactions using the fixture builder rather than connecting to the test dapp and creating transactions through the UI
```

3.

```javascript
// test file: test/e2e/tests/add-account.spec.js
scenario: should not affect public address when using secret recovery phrase to recover account with non-zero balance
solution: replace UI steps that build up extension state with the FixtureBuilder
```

4.

```javascript
// test file: test/e2e/tests/import-flow.spec.js
scenario: import Account using private key and remove imported account
solution: Replace UI steps that build up extension state with the FixtureBuilder
```

## Enhancing test stability with request mocking

By intercepting network requests and substituting responses with predefined mocks, we can significantly improve the speed and stability of our end-to-end tests by eliminating reliance on external services. This approach not only gives us greater control over APIs, enabling us to test a wide range of scenarios including network errors, but also helps us verify the extension's behaviour under adverse network conditions.
It's important to note that third-party websites and applications, which are beyond the control of the MetaMask engineering team, may not always behave consistently or be available. Relying on these external services introduces a dependency into our tests, increasing the chance of test failures due to factors outside our control, such as content changes, service issues, or unstable network connections. Therefore, we should aim to minimise such dependencies and be prepared for intermittent access issues.

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

### Proposal

Move mocking functions to a central location.

Current mock code in test:

```javascript
// test/e2e/helpers.js

setupPhishingDetectionMocks(mockServer, metamaskPhishingConfigResponse);

mockPhishingDetection(mockServer);
```

Proposed solution: Move both mocking functions to `test/e2e/mock-e2e.js`

### Guidelines for limiting third-party calls

By reducing reliance on external services and using techniques like response mocking and local simulations, we can control unpredictability and enhance test reliability.

- ✅ Full control over the dapp by using versions. No internet connection required.

```javascript
Type: E2E Test Dapp
Url: http://localhost:8080
```

- ⚠️ Full control over changes or updates to the dapp by using versions. Requires an internet connection

```javascript
Type: Test Snaps
Url: https://metamask.github.io/test-snaps/5.1.0/
```

- ⚠️ Partial control over changes or updates to the dapp. Requires an internet connection.

```javascript
Type: E2E Test Dapp
Url: https://metamask.github.io/test-dapp/
```

- ❌ No control over changes or updates to the dapp. Requires an internet connection.

```javascript
Type: Dapp
Url: https://app.uniswap.org/#/swap
```

### Proposal

Reduce the dependency on external sites by removing tests or mocking the dependency
Current external dependency in test:

```javascript
File: test/e2e/tests/import-flow.spec.js
Test Name: Connects to a Hardware wallet for Trezor
```

Proposed solution: The Trezor import flow involves opening the Trezor website, then the user takes additional steps on that website to connect the device. We can create a fake version of this website for testing, and update our test build to use the fake version. Investigate phishing detection solution, replacing Github.com with an empty page

## Test Atomicity and Smart Test Coupling

### Guidelines

Maintaining test atomicity and smart test coupling is crucial for effective end-to-end testing. Test atomicity ensures that each test is self-contained and independent, avoiding dependencies between tests. This is important because test coupling, where the outcome of one test influences the execution or success of another, can complicate the identification of failure causes.
To ensure test isolation and simplify parallelisation, each test is run with a dedicated browser and dedicated mock services (e.g. fixtures, metrics, etc.). We use the `withFixtures` function to create everything the test needs and to perform all cleanup steps after each test, ensuring that they are completely isolated from each other. We should avoid the use of shared mocks and services between tests.
On the other hand, allowing a degree of test coupling in E2E tests can enhance our testing effectiveness. This approach allows us to construct tests that combine multiple features, providing a closer approximation to real user behaviour. Unlike our E2E tests, real users do not commence each task with a 'clean' setup. Combining several atomic tests into a single user-flow based test may increase the execution time for individual tests. However, it eliminates the need for repeated setup, extension installation, and opening of the extension. As a result, the total test execution time is likely to decrease.

However, it's important to strike a balance. Over-reliance on long, combined tests can lead to slower, more complex tests that are harder to maintain. Also, if one step fails, the entire test fails, which can make it harder to pinpoint the issue.The key is to understand when and how tests should be isolated from each other, and when they can be strategically combined. Whether you choose to isolate or combine tests will depend on what you're trying to achieve with your tests.
Here are some guidelines to decide when to isolate or combine tests:

- Isolate tests for specific functionalities: If you're testing the behaviour of a single component or feature, it's best to isolate it from other parts of the application. This approach ensures that any test success or failure is due to the component under test, eliminating external factors.
- Combine tests for multiple user flows or cross-functional tests: When testing how different parts of the application work together to perform a more complicated user journey, combining tests can be beneficial. This approach allows you to verify a suite of user flows in a cohesive manner.
- Adopt a fail-fast philosophy: If an initial step in a test sequence fails, it may not be logical or efficient to proceed with subsequent steps. In such cases, adopting test coupling can be beneficial. However, consider the impact of a failure. If a failure in one part of a combined test makes it impossible to test the rest, but you still need to test the subsequent parts regardless of the outcome of the first part, it might be better to isolate the tests.

Remember, the goal is to create tests that are reliable, easy to understand, and provide valuable feedback about your system. Whether you choose to isolate or combine tests will depend on what you're trying to achieve within your tests. Ideally, we should aim for a large number of unit tests to test individual code pieces and a medium number of E2E tests for user flow testing that cover all the scenarios.
