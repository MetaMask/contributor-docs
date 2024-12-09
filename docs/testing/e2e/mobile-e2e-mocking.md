
# Mocking APIs in MetaMask Mobile for Enhanced E2E Testing

## Introduction
This document outlines how MetaMask Mobile uses API mocking to boost End-to-End (E2E) testing. Mocking lets us simulate different conditions that the app might face, ensuring it functions reliably across various scenarios.

## Mocking vs. E2E Testing
While E2E tests verify the overall user experience, mocking focuses on testing individual components. Here’s how they differ:

- **E2E Tests**: These validate the app's functionality as a whole, interacting with real APIs and backend services.
- **Mocking**: Simulates responses from APIs, allowing us to test components in specific network conditions or edge cases.

We use mocking to enhance our E2E testing, especially for scenarios where E2E alone might be unreliable or tricky to set up.

## File Structure
We keep E2E and mock tests separate with file naming conventions:

```plaintext
root/
├── e2e/
│   ├── spec/
│   │   ├── Accounts/
│   │   │   └── spec.js
                mock.spec.js# Mock test for Accounts
│   │   ├── Transactions/
│   │   │   └── spec.js
            mock.spec.js # Mock test for Transactions
│   ├── api-mocking/
│       ├── api-mocking/
│       │   ├── mock-responses/
│       │   │   ├── gas-api-responses.json
```

This structure promotes clear organisation and makes managing tests simpler.

## Mock Server Implementation

### Overview
We use Mockttp to simulate API responses, providing flexible testing across different HTTP methods. This allows us to test app behaviour even when external dependencies are unavailable or unreliable.

### Key Features
- Supports multiple HTTP methods (GET, POST, etc.)
- Configurable requests and responses
- Logs responses to simplify debugging

### Naming Mock Test Files
Mock test files are named with `.mock.spec.js` to keep things organised. For example, a test for the suggested gas API would be named:

`suggestedGasApi.mock.spec.js`

### Setting Up the Mock Server
The `startMockServer` function in `e2e/api-mocking/mock-server.js` starts the mock server. It takes events organised by HTTP methods, specifying the endpoint, response data, and request body (for POST requests).

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

### Defining Mock Events
`mockEvents.js` defines mock events, including:

- `urlEndpoint`: The API endpoint being mocked
- `response`: The mock response the server will return
- `requestBody`: Expected request body (for POST requests)

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

### Response Structure
Mock responses are stored in individual JSON files for each API or service within the `mock-responses` folder, making them easier to maintain and manage. Each API service has its own JSON response file, such as `gasApiResponse.json` for gas-related responses and `ethpriceResponse.json` for Ethereum price responses. This organisation enables clear separation of mock data and simplifies updates or additions.

**Example:** `gasApiResponse.json`

```json
{
  "suggestedGasApiResponses": {
    "error": {
      "message": "Internal Server Error"
    }
  },
  "suggestedGasFeesApiGanache": {
    # ... detailed gas fee data ...
  }
}
```

## Logging
The mock server logs response statuses and bodies to help track mocked requests, making debugging more straightforward.

## Using Mock Testing Effectively
Mock testing should work hand-in-hand with E2E testing. Here’s some guidance:

### When to Use Mocks:
- For testing isolated features without relying on live data
- For testing edge cases that are tricky to reproduce with real data
- For deterministic test results by controlling inputs and outputs

### When Not to Use Mocks:
- Overusing mocks may result in tests that don’t reflect real-world conditions.
- Integration with live services is essential for accurate testing.

### Exception for Complex Mock Events
For more complex mock events or criteria, you can use the `mockspecificTest` attribute to define custom mock events.

### Current Workaround for Network Request Interception
Due to limitations in intercepting network requests directly, we currently route traffic through a proxy server. This lets us intercept and mock requests as needed.

## Future Improvements
We’re looking into further enhancements for our mocking setup to simplify processes and improve test coverage.
