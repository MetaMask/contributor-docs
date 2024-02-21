# MetaMask E2E Test Guidelines

## General

The primary goal of writing end-to-end (E2E) tests at MetaMask is to ensure that all of the user flows within the product function as designed. Therefore, test coverage is a crucial metric for evaluating and improving quality. The higher the coverage is, the more confidence this creates among engineers in the product, and the more effectively bugs can be identified and eliminated.

The process of creating an E2E test involves several steps: identification, development, testing, and maintenance. Initially, it is important to identify the right aspects of the product to test. The next step is to develop the tests, adhering to the quality guidelines outlined in this document. This includes creating clear test cases, using appropriate assertions, and structuring the tests for easy maintenance. Finally, it's crucial to ensure that tests are robust and reliable. This means the test should consistently produce the same results and be resilient to minor system changes. By following these steps, effective E2E tests can be created that contribute to delivering a high-quality product.

This document serves as a guideline to approach writing E2E tests so that they provide fast feedback, they become less fragile over time, they are painless to debug, and they are scalable and easy to maintain. The remainder of this document will attempt to explain the suggested approach with relevant examples to help better understand the future direction for tests.

These guidelines aren't meant to be a strict set of rules, they should be thought of as an approach to writing robust end-to-end tests. Use them where they make sense and adapt them as necessary to complement your tests.


We have two distinct ways of structuring tests on extension and mobile  

You can read up on [Extension](e2e/extension-e2e-guidelines.md)
And [Mobile](e2e/extension-e2e-guidelines.md)


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
