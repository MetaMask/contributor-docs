# MetaMask E2E Test Guidelines

## General

The primary goal of writing end-to-end (E2E) tests at MetaMask is to ensure that all of the user flows within the product function as designed. Therefore, test coverage is a crucial metric for evaluating and improving quality. The higher the coverage is, the more confidence this creates among engineers in the product, and the more effectively bugs can be identified and eliminated.

The process of creating an E2E test involves several steps: identification, development, testing, and maintenance. Initially, it is important to identify the right aspects of the product to test. The next step is to develop the tests, adhering to the quality guidelines outlined in this document. This includes creating clear test cases, using appropriate assertions, and structuring the tests for easy maintenance. Finally, it's crucial to ensure that tests are robust and reliable. This means the test should consistently produce the same results and be resilient to minor system changes. By following these steps, effective E2E tests can be created that contribute to delivering a high-quality product.

This document serves as a guideline to approach writing E2E tests so that they provide fast feedback, they become less fragile over time, they are painless to debug, and they are scalable and easy to maintain. The remainder of this document will attempt to explain the suggested approach with relevant examples to help better understand the future direction for tests.

These guidelines aren't meant to be a strict set of rules, they should be thought of as an approach to writing robust end-to-end tests. Use them where they make sense and adapt them as necessary to complement your tests.

We have implemented two separate approaches for structuring tests on extensions and mobile. This document serves as a valuable reference point for establishing a common ground when writing end-to-end (E2E) tests across both clients. To gain a more comprehensive understanding of the specific guidelines for test writing on each platform, you can find detailed information here:

- [Extension](e2e/extension-e2e-guidelines.md)
- [Mobile](e2e/mobile-e2e-guidelines.md)

Below, we present our standardized approach to writing tests that applies universally to both clients

## Test names

The test name should communicate the purpose and behaviour of the test. A clear test name serves as a concise summary of what the test aims to verify, making it easier for everyone to understand and maintain the tests. Clear test names improve the readability of the test and help when it comes to debugging a failed test. You should be able to figure out the purpose of the test without going through the complete implementation. If the test is failing, it should be easy to figure out which functionality is broken from the test name.

### Guidelines

✅ Recommended readable test names

```javascript
- adds Bob to the address book
- send 1 TST to Bob
```

❌ Test name should be completely avoided: Long test names that may over explain the test intention and the word `and` can decrease the readability of the test,making it harder to understand what the test is doing as well as diagnose and fix issues.

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
// in file: e2e/tests/add-account.spec.js
- should be possible to remove an account imported with a private key, but should not be possible to remove an account generated from the SRP imported in onboarding

// in file: e2e/tests/lockdown.spec.js
- the UI and background environments are locked down
```

## Organization of test files

It's essential to keep test files organised. As test suites get bigger, a well-structured organisation makes it easier to search for tests as well as identify logical groups of tests that may be impacted by changes to an area of the application. When tests are organised based on related features or functionality it becomes easier to identify common helper functions that can be shared across the tests, reducing duplication.

Organise tests into folders based on scenarios and features. This means that each type of scenario has its own folder, and each feature team owns one or more folders.

- This approach assigns ownership of E2E testing at the feature team level. Each feature team, being aware of the features they own, finds it easier to understand the existing tests and identify what's missing.
- The future goal is for specific feature teams to manage the necessary test cases for their own features, instead of the core teams tracking each one.
- Using this organizational strategy eliminates the need to decide at a low level where to place a test, as it is straightforward based on the feature or scenario.
- The extension and mobile team both adhere to the same method for organizing test files. This consistency reduces friction when switching context.

Example for organization of test files by features and scenarios:

```javascript
// current test path:
e2e/tests/nft/import-erc1155.spec.js
// recommended test path: (consolidate all import tests for different tokens into a single repository)
e2e/tests/tokens/import/import-erc1155.spec.js

// current test path:
e2e/tests/clear-activity.spec.js
// recommended test path:
e2e/tests/settings/clear-activity.spec.js

// current test path:
e2e/tests/ppom-blockaid-alert-erc20-approval.spec.js
// recommended test path:
e2e/tests/ppom/ppom-blockaid-alert-erc20-approval.spec.js
```

## Test Atomicity and Smart Test Coupling

### Guidelines

Maintaining test atomicity and smart test coupling is crucial for effective end-to-end testing. Test atomicity ensures that each test is self-contained and independent, avoiding dependencies between tests. This is important because test coupling, where the outcome of one test influences the execution or success of another, can complicate the identification of failure causes.

To ensure test isolation and simplify parallelisation, each test is run with a dedicated browser and dedicated mock services (e.g. fixtures, metrics, etc.). The `withFixtures` function helper can be used to create everything the test needs and to perform all cleanup steps after each test, ensuring that they are completely isolated from each other. It is best to avoid shared mocks and services between tests.

On the other hand, allowing a degree of test coupling in E2E tests can enhance the effectiveness of testing. This approach is useful to construct tests that combine multiple features, providing a closer approximation to real user behaviour. Unlike in E2E tests, real users do not commence each task with a 'clean' setup. Combining several atomic tests into a single user-flow based test may increase the execution time for individual tests. However, it eliminates the need for repeated setup, application installation, and opening of the application. As a result, the total test execution time is likely to decrease.

However, it's important to strike a balance. Over-reliance on long, combined tests can lead to slower, more complex tests that are harder to maintain. Also, if one step fails, the entire test fails, which can make it harder to pinpoint the issue. The key is to understand when and how tests should be isolated from each other, and when they can be strategically combined. Whether you choose to isolate or combine tests will depend on what you're trying to achieve with your tests.

Here are some guidelines to decide when to isolate or combine tests:

- Isolate tests for specific functionalities: If you're testing the behaviour of a single component or feature, it's best to isolate it from other parts of the application. This approach ensures that any test success or failure is due to the component under test, eliminating external factors.
- Combine tests for multiple user flows or cross-functional tests: When testing how different parts of the application work together to perform a more complicated user journey, combining tests can be beneficial. This approach allows you to verify a suite of user flows in a cohesive manner.
- Adopt a fail-fast philosophy: If an initial step in a test sequence fails, it may not be logical or efficient to proceed with subsequent steps. In such cases, adopting test coupling can be beneficial. However, consider the impact of a failure. If a failure in one part of a combined test makes it impossible to test the rest, but you still need to test the subsequent parts regardless of the outcome of the first part, it might be better to isolate the tests.

Remember, the goal is to create tests that are reliable, easy to understand, and provide valuable feedback about the system. Whether you choose to isolate or combine tests will depend on what you're trying to achieve within your tests. Ideally, you should aim for a large number of unit tests to test individual code pieces and a medium number of E2E tests for user flow testing that cover all the scenarios.

## Controlling state

To achieve test atomicity and ensure E2E tests are stable and reliable, it is best to control the state of the application programmatically rather than relying on the application UI. Doing so eliminates unnecessary UI interactions, decreasing the amount of possible breaking points in a test. It also improves test stability by minimising issues caused by timing synchronisation or inconsistencies in the UI, reducing the test execution time and allowing the test to provide fast and focused feedback.

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
solution: replace UI steps that build up the application state with the FixtureBuilder
```

4.

```javascript
// test file: test/e2e/tests/import-flow.spec.js
scenario: import Account using private key and remove imported account
solution: replace UI steps that build up the application state with the FixtureBuilder
```
