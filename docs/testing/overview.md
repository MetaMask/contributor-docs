# Testing Overview

At MetaMask, we are committed to delivering high-quality software. These guidelines describe how we use automated tests to maximize product quality while minimizing the cost of writing and maintaining tests.

Our automated tests fall into three general categories: end-to-end (e2e), integration and unit tests. This document provides comprehensive guidelines on each testing type, emphasizing the importance of choosing the right test for the right scenario and detailing the specific practices for writing and organizing these tests.

## Testing principles

What makes a good test? Here are some general principles.

### 1. Tests should describe expectations

What's the difference between a bug and a feature? A feature is expected, a bug is not.

If it's not clear what the code-under-test is supposed to do, it won't be clear what to do when the test fails either. Clear expectations are essential for an effective test.

### 2. Tests should catch bugs

This is the main purpose of tests. Take care to ensure that each test actually fails when the described expectation is not met.

### 3. Tests should not break often

It's expected that tests will break when we change the behavior or APIs under test. But if a test breaks often from refactors, or from changes to behavior or APIs not directly under test, it's sign that the test may be poorly designed.

Test breakages are expensive. We should do what we can to minimize them.

### 4. Tests should be easy to read

A test isn't effective if we can't understand why it failed. The more readable a test is, the faster we can fix it.

### 5. Tests should be easy to write

Some behaviors are inherently more complex than others, increasing the difficulty of testing them. But in general we want tests to be easy to write, so that we can keep costs low.

If a test is hard to write, consider that as a warning that you might be approaching the problem in a non-optimal way. Resist the urge to power through the work, and instead consider how redesigning the code-under-test or using a different testing strategy might save time.

### 6. Tests should be fast

Making tests faster has a dramatic improvement on productivity, reducing feedback loops.

What we consider "fast" depends on what the test does; what's fast for an end-to-end test might be slow for a unit test. But faster is always better, for all types of tests.

### 7. Tests should have a high return on investment

If a test doesn't align with the previous 6 principles, it's probably doing more harm than good. Consider deleting it.

That doesn't mean we should skip writing tests with it's too difficult. On the contrary; we have high expectations for quality, we don't have the luxury of writing only the easiest most valuable tests. We need to aim for total coverage.

But writing and maintaining bad tests is not the shortest path to complete test coverage. Our time is better spent writing good tests, and making investments that allow us to write more good tests. All code can be designed to be easy to test, and even the most complex test setup/teardown can be reasonably fast and easy to read/write with the right tools.

## What are unit tests?

Unit testing is a conceptually vague practice, as there is no consensus on how they should be written and what they should test. But in general, [as outlined by Martin Fowler](https://martinfowler.com/articles/2021-test-shapes.html), there are two common approaches: solitary unit tests and sociable unit tests. At MetaMask, we employ both of these types of unit tests (see [discussion on the use cases of each type](https://github.com/MetaMask/core/pull/3827#discussion_r1469377179)).
Unit tests examine individual components or functions in isolation or within their immediate context. At MetaMask we encourage a flexible approach to unit testing, recognizing the spectrum between sociable and solitary unit tests.

### When to write unit tests?

All components or functions should be validated through unit tests. These tests focus on the component's or function's internal logic and functionality.
If UI integration tests are available, those are preferred for page-level components. However, developers may choose to write sociable unit tests for specific scenarios where UI integration tests might not be necessary or practical.

**Best Practices**: Follow the [unit test best practices](./unit-testing.md).

## What are UI integration tests?

UI integration tests evaluate the interaction between multiple components within the application, ensuring they work together as intended. For MetaMask, we are aiming to make UI integration tests particularly focused on page-level components and are conducted in the context of the full UI application.

### When to write UI integration tests?

UI integration tests should be written for page-level components. These tests should simulate user journeys and validate the application's behavior in a full app context.

**Best Practices**: Follow the [ui integration tests best practices](./ui-integration-testing.md).

## What are end-to-end tests?

End-to-end tests simulate real user scenarios from start to finish, testing the application as a whole. In the Extension, end-to-end tests are tests that strive to test a real extension app (including `background`, `contentscript` and `ui` processes) in a controlled environment from a user perspective.

### When to write end-to-end tests?

End-to-end tests exercise an application's integration with external systems or services through critical user journeys. This includes any interactions between the UI, background process, dapp, blockchain, etc. End-to-end tests should closely mimic real user actions and flows.

**Best Practices**: Follow the [end-to-end test best practices](./e2e-testing.md).

## Generic testing considerations

- **Code fencing:** Code fencing is available in MetaMask Extension and Mobile codebases. Though useful for its purposes, this practice is very problematic for unit/UI integration testing. We should avoid it.

## Conclusion

To sum up, understanding the distinction between end-to-end, UI integration and unit tests and applying them appropriately is crucial for maintaining MetaMask's code quality and reliability. UI integration tests offer a broad view of user interactions and system integration for page-level components, while unit tests provide detailed insights into the functionality of individual components, and end-to-end tests validate the application's overall behavior, ensuring readiness for production.
By following these guidelines, developers can ensure comprehensive test coverage, contributing to the robustness and user satisfaction of the application
