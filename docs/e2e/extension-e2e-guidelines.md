# MetaMask Extension E2E Test Guidelines

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

It's essential to keep test files organised. As test suites get bigger, a well-structured organisation makes it easier to search for tests as well as identify logical groups of tests that may be impacted by changes to an area of the extension. When tests are organised based on related features or functionality it becomes easier to identify common helper functions that can be shared across the tests, reducing duplication.

Organise tests into folders based on scenarios and features. This means that each type of scenario has its own folder, and each feature team owns one or more folders.

- This approach assigns ownership of E2E testing at the feature team level. Each feature team, being aware of the features they own, finds it easier to understand the existing tests and identify what's missing.
- The future goal is for specific feature teams to manage the necessary test cases for their own features, instead of the core teams tracking each one.
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

Crafting resilient locators is crucial for reliable tests. It’s important to write selectors that are resilient to changes in the extension's UI. Tests become less prone to issues as the extension is updated, for example, during UI redesigns. This results in less effort required to maintain and update the tests, improving the stability of the tests and reducing the associated maintenance costs. Element locators should be independent of CSS or JS so that they do not break on the slightest UI change. Another consideration is whether a test should fail if the content of an element changes.

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

## Controlling state

To achieve test atomicity and ensure E2E tests are stable and reliable, it is best to control the state of the extension programmatically rather than relying on the application UI. Doing so eliminates unnecessary UI interactions, decreasing the amount of possible breaking points in a test. It also improves test stability by minimising issues caused by timing synchronisation or inconsistencies in the UI, reducing the test execution time and allowing the test to provide fast and focused feedback.

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

Here are some examples to remove redundant steps following above guidelines:

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
solution: replace UI steps that build up extension state with the FixtureBuilder
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