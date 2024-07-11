# Test guidelines

At MetaMask, we are committed to delivering high-quality software. A key aspect of achieving this goal is through a robust testing strategy that includes both end-to-end (e2e), integration and unit tests. This document provides comprehensive guidelines on each testing type, emphasizing the importance of choosing the right test for the right scenario and detailing the specific practices for writing and organizing these tests.

## What are unit tests?

Unit tests understanding is a bit more diffuse. As there is no consensus on how they should be written and what they should test. But generically, [as captured by Martin Fowler](https://martinfowler.com/articles/2021-test-shapes.html), there's two lines of thought: solitary unit tests and sociable unit tests. Within MetaMask there are multiple examples from tests and discussions highlighting the two types (for example you can check the discussion on [this PR](https://www.google.com/url?q=https://github.com/MetaMask/core/pull/3827%23discussion_r1469377179&sa=D&source=editors&ust=1716475694980436&usg=AOvVaw3oTcfVgfRuiQFSDChZPrFM))
Unit tests examine individual components or functions in isolation or within their immediate context. At MetaMask we encourage a flexible approach to unit testing, recognizing the spectrum between sociable and solitary unit tests.

**When to write unit tests?**
All components, except page-level components, should be validated through unit tests. These tests focus on the component's internal logic and functionality. Integration tests are preferred for page-level components. However, developers may choose to write sociable unit tests for specific scenarios where integration tests might not be necessary or practical.

**Generic guidelines**

- **Flexibility in Isolation**: Depending on the test's purpose, developers can choose between sociable unit tests (without mocking child components) and solitary unit tests (with mocked dependencies) to best suit the testing needs.
- **Colocation**: Keep unit test files next to their corresponding implementation files. This practice enhances test discoverability and maintenance.
- **Granularity and Focus**: Ensure unit tests are focused and granular, targeting specific functionalities. The choice between sociable and solitary testing should be guided by the test's objectives and the component's role within the application.
- **Unit Tests for Page Components**: While integration tests are the primary method for testing page-level components, developers have the discretion to use unit tests for specific cases. This flexibility allows for targeted testing of component logic or behavior that may not require the full app context.
- **Best Practices**: Follow the [unit test best practices](./unit-testing.md).

## What are integration tests?

Integration tests evaluate the interaction between multiple components within the application, ensuring they work together as intended. For MetaMask, we are aiming to make UI integration tests particularly focused on page-level components and are conducted in the context of the full UI application.

**When to write integration tests?**
Integration tests should be written for page-level components. These tests should simulate user journeys and validate the application's behavior in a full app context.

**Generic guidelines**

- **Full App Context**: Always test page-level components in the context of the full application. This approach ensures that tests reflect real user scenarios and interactions within the app.
- **Focus on User Journeys**: Design tests to mimic actual user actions and flows. This helps in identifying issues that could affect the user experience.
- **Avoid Testing Non-Page Components**: Integration tests should not be written for components other than page-level components. Other components should be tested using unit tests.
- **Avoid Snapshots**: Refrain from using snapshot testing in integration tests. They tend to be less meaningful in the context of full app testing and can lead to brittle tests.
- **Test Location**: Place integration test files within a `test/integration` directory. This centralized location helps manage tests that span multiple pages and components, improving maintenance and discoverability. This approach differs from unit tests, where co-location with the component is standard.
- **No Mocks**: Keep mocking to minimum. Ideally only the background connection, or any network request (fired from the UI) should be mocked.

## What are e2e tests?

e2e tests simulate real user scenarios from start to finish, testing the application as a whole. In the Extension, e2e tests are tests that strive to test a real extension app (including background, contentscript and ui processes) in a controlled environment from a user perspective.

**When to write e2e tests?**
Testing application's integration with external systems or services through critical user journeys. This includes any interactions between the UI, background process, dapp, blockchain, etc.

**Generic guidelines**

- **Realistic Scenarios**: Test scenarios that closely mimic real user actions and flows.
- **Best Practices**: Follow the [e2e test best practices](./e2e-testing.md).

## Broader guidelines

- **Code fencing:** is very problematic for unit/integration testing. We should avoid it.
- **jest manual mocks:** This types of mocks are automatically picked up by jest for all tests. We should be very careful when writing this types of mocks as they will be shared across all tests. And with integration tests, as we are aiming to mock as little as possible, we should avoid this at all costs.

## Conclusion

To sum up, understanding the distinction between e2e, integration and unit tests and applying them appropriately is crucial for maintaining MetaMask's code quality and reliability. Integration tests offer a broad view of user interactions and system integration for page-level components, while unit tests provide detailed insights into the functionality of individual components,and e2e tests validate the application's overall behavior, ensuring readiness for production.
By following these guidelines, developers can ensure comprehensive test coverage, contributing to the robustness and user satisfaction of the application
