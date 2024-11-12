# UI Integration Testing Guidelines

## Use Jest

Just like for unit tests, for UI integration tests, we use Jest as the test runner and testing framework. Jest provides a rich set of features that make it easy to write and maintain tests. UI integration tests should be written following the same conventions as unit tests.

## Focus on User Journeys

UI integration tests should focus on user journeys and interactions within the application. Design tests to mimic actual user actions and flows. This helps in identifying issues that could affect the user experience.

### Provide Full App Context

Always test page-level components in the context of the full application. This approach ensures that tests reflect real user scenarios and interactions within the app.

UI integration tests should not be written for components other than page-level components. Other components should be tested using unit tests.

## Keep Tests Separate from Implementation

Place integration test files within a `test/integration` directory. This centralized location helps manage tests that span multiple pages and components, improving maintenance and discoverability. This approach differs from unit tests, where co-location with the component is standard.

## Avoid Snapshot testing

Refrain from using snapshot testing in UI integration tests. They tend to be less meaningful in the context of full app testing and can lead to brittle tests.

## Don't Mock UI Components

Keep mocking to minimum. Ideally only the background connection (MetaMask Extension), or any network request (fired from the UI) should be mocked. Avoid mocking any UI components or hooks. For mocking the background connection we can use jest, for mocking network requests we can use `nock` (or `msw`).
