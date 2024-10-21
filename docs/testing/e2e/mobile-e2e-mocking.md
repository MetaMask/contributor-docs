# Mocking APIs in MetaMask Mobile for Enhanced E2E Testing

## Mocking

Mocking is an essential technique in our testing strategy, allowing us to simulate various conditions the application may encounter. By creating mock responses for API endpoints, we can effectively test how the app behaves in scenarios like network failures or receiving unexpected data.

We view mocking as a complement to, not a replacement for, end-to-end (E2E) testing. While E2E tests provide confidence in the overall functionality of the MetaMask app, mocking enables focused component tests that help identify issues early in development. This ensures the app can handle different states gracefully.

E2E tests should be used wherever possible to validate the application for its end users, while mocking fills gaps where E2E tests may be unreliable or difficult to set up. As more mocked tests are added, our implementation strategy will evolve, keeping testing relevant and effective.

## File Structure

We maintain a clear and organised structure to separate E2E tests from mocked tests, which helps manage our testing efforts effectively.

```plaintext
root/
├── e2e/
│   ├── spec/
│   │   ├── Accounts/
│   │   │   └── spec.js
│   │   ├── Transactions/
│   │   │   └── spec.js
│   ├── mocked-specs/
│   │   ├── Accounts/
│   │   │   └── mockSpec.js
│   │   ├── Transactions/
│   │       └── mockSpec.js
```

This structure allows us to keep E2E tests focused on overall app functionality while leveraging mocks to simulate various conditions.

## Mock Server Implementation Guide

### Overview

This guide outlines how to implement API request mocking using Mockttp for mobile testing. Mocking lets you simulate API responses and handle both GET and POST requests during testing, ensuring the app behaves correctly even when external dependencies are unavailable or unreliable.

### Key Features

- Handles both GET and POST requests.
- Configurable requests and responses through events, allowing flexibility in response statuses and request bodies.
- Logs responses for easier debugging during tests.

### Setting Up the Mock Server

To start the mock server, use the `startMockServer` function from `e2e/mockServer/mockServer.js`. This function accepts events organised by HTTP methods (GET, POST), specifying the endpoint, the response to return, and the request body for POST requests.

```javascript
import { mockEvents } from '../mockServer/mock-config/mock-events';

mockServer = await startMockServer({
  GET: [
    mockEvents.GET.suggestedGasApiErrorResponse,
  ],
  POST: [
    mockEvents.POST.suggestedGasApiPostResponse,
  ],
});
```

This starts the mock server and ensures it listens for the defined routes, returning the specified responses.

### Defining Mock Events

Mock events are defined in the `e2e/mockServer/mock-config/mock-events.js` file. Each key represents an HTTP method, and events include:

- **urlEndpoint**: The API endpoint being mocked.
- **response**: The mock response the server will return.
- **requestBody**: (For POST requests) The expected request body.

```javascript
export const mockEvents = {
  GET: {
    suggestedGasApiErrorResponse: {
      urlEndpoint: 'https://gas.api.cx.metamask.io/networks/1/suggestedGasFees',
      response: { status: 500, message: 'Internal Server Error' },
    },
  },
  POST: {
    suggestedGasApiPostResponse: {
      urlEndpoint: 'https://gas.api.cx.metamask.io/networks/1/suggestedGasFees',
      response: { status: 200, message: 'Success' },
      requestBody: { priorityFee: '2', maxFee: '2.000855333' },
    },
  },
};
```

The mock responses are stored in `e2e/mockServer/mock-responses/mockResponses.js`, keeping the responses modular and reusable.

### Naming Test Files

Mock test files should include `.mock.spec.js` in their filename for clarity and organisation. For example, tests for the suggested gas API would be named:

```
suggestedGasApi.mock.spec.js
```

This naming convention makes it easy to distinguish mock tests from E2E tests, maintaining a clean and organised test suite.

### Logging

The mock server logs the response status and body to help track what’s being mocked, making debugging simpler.

### Strategy for Using Mock Testing

Mock testing should complement E2E testing. The aim is to use E2E tests to gain confidence in the app's functionality wherever possible. However, mock testing can be used to fill gaps in scenarios where E2E tests may be unreliable or challenging, such as:

- Testing components under specific network conditions.
- Ensuring consistent behaviour across mobile platforms.
- Simulating server errors or slow responses.

By blending E2E and mock testing, we ensure comprehensive test coverage while maintaining fast and reliable tests that simulate real-world conditions.

## File Structure

Here’s an example of how your files should be structured:

- `e2e/mockServer/mockServer.js`: The main file where the mock server is started.
- `e2e/mockServer/mock-config/mock-events.js`: Defines mock events (GET and POST) and their associated endpoints and responses.
- `e2e/mockServer/mock-responses/mockResponses.js`: Contains the mock responses used by the mock events.

Following this structure and strategy ensures efficient mock testing, complementing your E2E tests.
