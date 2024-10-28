
# Mocking APIs in MetaMask Mobile for Enhanced E2E Testing

## Mocking

Mocking is an essential technique in our testing strategy, allowing us to simulate various conditions the application may encounter. By creating mock responses for API endpoints, we can effectively test how the app behaves in scenarios like network failures or receiving unexpected data.

We view mocking as a complement to, not a replacement for, end-to-end (E2E) testing. While E2E tests provide confidence in the overall functionality of the MetaMask app, mocking enables focused component tests that help identify issues early in development. This ensures the app can handle different states gracefully.

E2E tests should be used wherever possible to validate the application for its end users, while mocking fills gaps where E2E tests may be unreliable or difficult to set up. As more mocked tests are added, our implementation strategy will evolve, keeping testing relevant and effective.

## File Structure

To identify mocked tests, we rely on naming conventions for now. We maintain a clear and organised structure to separate E2E tests from mocked tests, which helps manage our testing efforts effectively.


```plaintext
root/
├── e2e/
│   ├── spec/
│   │   ├── Accounts/
│   │   │   └── spec.js
                mock.spec.js
│   │   ├── Transactions/
│   │   │   └── spec.js
            mock.spec.js
│   ├── api-mocking/
│       ├── api-mocking/
│       │   ├── mock-responses/
│       │   │   ├── gas-api-responses.json
```


This structure allows us to keep E2E tests focused on overall app functionality while leveraging mocks to simulate various conditions.


## Mock Server Implementation Guide

### Overview

This guide outlines how to implement API request mocking using Mockttp for mobile testing. Mocking lets you simulate API responses and handle any HTTP method required during testing, ensuring the app behaves correctly even when external dependencies are unavailable or unreliable.

### Key Features

- Handles multiple HTTP methods as required by teams.
- Configurable requests and responses through events, allowing flexibility in response statuses and request bodies.
- Logs responses for easier debugging during tests.

### Naming Test Files

Mock test files should include `.mock.spec.js` in their filename for clarity and organisation. For example, tests for the suggested gas API would be named:

```
suggestedGasApi.mock.spec.js
```

This naming convention makes it easy to distinguish mock tests from E2E tests, maintaining a clean and organised test suite.

### Setting Up the Mock Server

To start the mock server, use the `startMockServer` function from `e2e/api-mocking/mock-server.js`. This function accepts events organised by HTTP methods (e.g., GET, POST), specifying the endpoint, the response to return, and the request body for POST requests. The function enables us to pass multiple events enabling us to mock multiple services at once

```javascript
import { mockEvents } from '../api-mocking/mock-config/mock-events';

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

Mock events are defined in the `e2e/api-mocking/mock-config/mock-events.js` file. Each key represents an HTTP method, and events include:

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

The mock responses are stored in a separate JSON file `mockResponses.json`, located in `e2e/api-mocking/mock-responses/`, keeping the responses modular and reusable.

### Response Structure in `mockResponses.json`

# Mock Response Structure

Mock responses are now organised into individual JSON files for each related API or service, simplifying access and maintenance. Each file contains identifiable keys specific to the API, with subkeys as needed to structure various response scenarios, such as suggestedGasApiResponses or suggestedGasFeesApiGanache.

### Example: `gas-api-responses.json`

```json
{
  "suggestedGasApiResponses": {
    "error": {
      "message": "Internal Server Error"
    }
  },
  "suggestedGasFeesApiGanache": {
    "low": {
      "suggestedMaxPriorityFeePerGas": "1",
      "suggestedMaxFeePerGas": "1.000503137",
      "minWaitTimeEstimate": 15000,
      "maxWaitTimeEstimate": 60000
    },
    "medium": {
      "suggestedMaxPriorityFeePerGas": "1.5",
      "suggestedMaxFeePerGas": "1.500679235",
      "minWaitTimeEstimate": 15000,
      "maxWaitTimeEstimate": 45000
    },
    "high": {
      "suggestedMaxPriorityFeePerGas": "2",
      "suggestedMaxFeePerGas": "2.000855333",
      "minWaitTimeEstimate": 15000,
      "maxWaitTimeEstimate": 30000
    },
    "estimatedBaseFee": "0.000503137",
    "networkCongestion": 0.34,
    "latestPriorityFeeRange": ["1.5", "2"],
    "historicalPriorityFeeRange": ["0.000001", "236.428872442"],
    "historicalBaseFeeRange": ["0.000392779", "0.00100495"],
    "priorityFeeTrend": "up",
    "baseFeeTrend": "up",
    "version": "0.0.1"
  }
}



### Logging

The mock server logs the response status and body to help track what’s being mocked, making debugging simpler.

### Strategy for Using Mock Testing

Mock testing should complement E2E testing. The aim is to use E2E tests to gain confidence in the app's functionality wherever possible. However, mock testing can be used to fill gaps in scenarios where E2E tests may be unreliable or challenging, such as:

- Testing components under specific network conditions.
- Ensuring consistent behaviour across mobile platforms.
- Simulating server errors or slow responses.

By blending E2E and mock testing, we ensure comprehensive test coverage while maintaining fast and reliable tests that simulate real-world conditions.

## When to Use Mocks

You should use mocks in scenarios such as testing isolated features without relying on live data or real backend services. This includes testing edge cases that are difficult to reproduce with real data or ensuring deterministic test results by controlling the inputs and outputs. For example, when the `suggestedGasApi` is down, the app should default to the legacy modal and API. This is a scenario that cannot be consistently tested with E2E or even manually. Mocking enables us to test the app's behaviour in isolated scenarios or edge cases that are difficult to reproduce efficiently in E2E or manual testing.

## When Not to Use Mocks

Be cautious against overusing mocks, especially when integration with real services is essential for accurate testing. Relying too heavily on mocks could result in tests that do not reflect real-world conditions, leading to false confidence in system stability.

## Exception for Complex Mock Events

Teams with complex mock events or criteria can utilise the `mockspecificTest` attribute, where you can define custom mock events in a separate instance to fit your unique requirements. This can be liased with the mobile QA platform team.
