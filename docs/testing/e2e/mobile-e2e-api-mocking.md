# MetaMask Mobile E2E API Mocking Guide

This guide covers the comprehensive API mocking system used in MetaMask Mobile E2E tests. For framework usage, see the [E2E Framework Guide](./mobile-e2e-framework-guide.md).

## 🎭 Mocking System Overview

MetaMask Mobile E2E tests use a sophisticated API mocking system that:

- **Intercepts all network requests** through a proxy server
- **Provides default mocks** for common APIs across all tests
- **Supports test-specific mocks** for custom scenarios
- **Handles feature flag mocking** for controlled feature testing
- **Tracks unmocked requests** for comprehensive coverage

## 🏗️ Mocking Architecture

```
e2e/api-mocking/
├── mock-responses/
│   ├── defaults/                    # Default mocks for all tests
│   │   ├── index.ts                # Aggregates all default mocks
│   │   ├── accounts.ts             # Account-related API mocks
│   │   ├── price-apis.ts           # Price feed mocks
│   │   ├── swap-apis.ts            # Swap/exchange API mocks
│   │   ├── token-apis.ts           # Token metadata mocks
│   │   ├── staking.ts              # Staking API mocks
│   │   └── user-storage.ts         # User storage service mocks
│   ├── feature-flags-mocks.ts      # Predefined feature flag configs
│   └── simulations.ts              # Transaction simulation mocks
├── helpers/
│   ├── mockHelpers.ts              # Core mocking utilities
│   └── remoteFeatureFlagsHelper.ts # Feature flag mocking helper
└── mock-server.ts                  # Mock server implementation
```

## 🔧 Default Mocks System

### How Default Mocks Work

Default mocks are automatically loaded for all tests via `FixtureHelper.createMockAPIServer()`:

1. **Mock server starts** on dedicated port
2. **Default mocks loaded** from [`e2e/api-mocking/mock-responses/defaults/`](https://github.com/MetaMask/metamask-mobile/tree/main/e2e/api-mocking/mock-responses/defaults)
3. **Test-specific mocks applied** (take precedence over defaults)
4. **Feature flags mocked** with default configurations
5. **Notification services mocked** automatically

### Default Mock Categories

Default mocks are organized by API category:

```typescript
// From e2e/api-mocking/mock-responses/defaults/index.ts
export const DEFAULT_MOCKS = {
  GET: [
    ...(ACCOUNTS_MOCKS.GET || []),
    ...(PRICE_API_MOCKS.GET || []),
    ...(SWAP_API_MOCKS.GET || []),
    ...(TOKEN_API_MOCKS.GET || []),
    ...(STAKING_MOCKS.GET || []),
    ...(USER_STORAGE_MOCKS.GET || []),
    // ... other categories
  ],
  POST: [
    ...(ACCOUNTS_MOCKS.POST || []),
    ...(SWAP_API_MOCKS.POST || []),
    // ... other categories
  ],
};
```

j

### Adding New Default Mocks

To add default mocks that benefit all tests:

1. **Create or edit a category file** in `defaults/` folder:

```typescript
// e2g. defaults/my-new-service.ts
import { MockApiEndpoint } from '../../framework/types';

export const MY_SERVICE_MOCKS = {
  GET: [
    {
      urlEndpoint: 'https://api.myservice.com/data',
      responseCode: 200,
      response: { success: true, data: [] },
    },
  ] as MockApiEndpoint[],
  POST: [
    {
      urlEndpoint: 'https://api.myservice.com/submit',
      responseCode: 201,
      response: { id: '123', status: 'created' },
    },
  ] as MockApiEndpoint[],
};
```

2. **Add to the main index file**:

```typescript
// defaults/index.ts
import { MY_SERVICE_MOCKS } from './my-new-service';

export const DEFAULT_MOCKS = {
  GET: [
    // ... existing mocks
    ...(MY_SERVICE_MOCKS.GET || []),
  ],
  POST: [
    // ... existing mocks
    ...(MY_SERVICE_MOCKS.POST || []),
  ],
};
```

## 🎯 Test-Specific Mocks

### Method 1: Using testSpecificMock Parameter

The most common approach for custom mocks:

```typescript
import { withFixtures } from '../framework/fixtures/FixtureHelper';
import FixtureBuilder from '../framework/fixtures/FixtureBuilder';
import { Mockttp } from 'mockttp';
import { setupMockRequest } from '../api-mocking/helpers/mockHelpers';

describe('Custom API Test', () => {
  it('handles custom API response', async () => {
    const testSpecificMock = async (mockServer: Mockttp) => {
      await setupMockRequest(mockServer, {
        requestMethod: 'GET',
        url: 'https://api.example.com/custom-endpoint',
        response: { customData: 'test-value' },
        responseCode: 200,
      });
    };

    await withFixtures(
      {
        fixture: new FixtureBuilder().build(),
        testSpecificMock,
      },
      async () => {
        // Test implementation that uses the custom mocked API
      },
    );
  });
});
```

### Method 2: Direct Mock Server Access

Access the mock server directly within tests:

```typescript
await withFixtures(
  {
    fixture: new FixtureBuilder().build(),
  },
  async ({ mockServer }) => {
    // Set up additional mocks within the test
    await setupMockRequest(mockServer, {
      requestMethod: 'POST',
      url: 'https://api.example.com/dynamic-endpoint',
      response: { result: 'success' },
      responseCode: 201,
    });

    // Test implementation
  },
);
```

## 🛠️ Mock Helper Functions

### setupMockRequest

For simple HTTP requests with various methods:

```typescript
import { setupMockRequest } from '../api-mocking/helpers/mockHelpers';

await setupMockRequest(mockServer, {
  requestMethod: 'GET', // 'GET' | 'POST' | 'PUT' | 'DELETE'
  url: 'https://api.example.com/endpoint',
  response: { data: 'mock-response' },
  responseCode: 200,
});

// URL patterns with regex
await setupMockRequest(mockServer, {
  requestMethod: 'GET',
  url: /^https:\/\/api\.metamask\.io\/prices\/[a-z]+$/,
  response: { price: '100.50' },
  responseCode: 200,
});
```

### setupMockPostRequest

For POST requests with request body validation:

```typescript
import { setupMockPostRequest } from '../api-mocking/helpers/mockHelpers';

await setupMockPostRequest(
  mockServer,
  'https://api.example.com/validate',
  {
    method: 'transfer',
    amount: '1000000000000000000',
    to: '0x742d35cc6634c0532925a3b8d0ea4405abf5adf3',
  }, // Expected request body
  { result: 'validated', txId: '0x123...' }, // Mock response
  {
    statusCode: 200,
    ignoreFields: ['timestamp', 'nonce'], // Fields to ignore in validation
  },
);
```

### Advanced Mock Patterns

```typescript
// Mock with custom headers
await setupMockRequest(mockServer, {
  requestMethod: 'GET',
  url: 'https://api.example.com/authenticated',
  response: { data: 'secure-data' },
  responseCode: 200,
  headers: {
    Authorization: 'Bearer mock-token',
    'Content-Type': 'application/json',
  },
});

// Mock error responses
await setupMockRequest(mockServer, {
  requestMethod: 'POST',
  url: 'https://api.example.com/failing-endpoint',
  response: { error: 'Internal Server Error' },
  responseCode: 500,
});
```

## 🚩 Feature Flag Mocking

### setupRemoteFeatureFlagsMock Helper

Feature flags control application behavior and must be mocked consistently:

```typescript
import { setupRemoteFeatureFlagsMock } from '../api-mocking/helpers/remoteFeatureFlagsHelper';

const testSpecificMock = async (mockServer: Mockttp) => {
  await setupRemoteFeatureFlagsMock(
    mockServer,
    {
      rewards: true,
      confirmation_redesign: {
        signatures: true,
        transactions: false,
      },
      bridgeConfig: {
        support: true,
        refreshRate: 5000,
      },
    },
    'main', // distribution: 'main' | 'flask'
  );
};
```

### Predefined Feature Flag Configurations

Use existing configurations from [`e2e/api-mocking/mock-responses/feature-flags-mocks.ts`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/api-mocking/mock-responses/feature-flags-mocks.ts):

```typescript
import {
  oldConfirmationsRemoteFeatureFlags,
  confirmationsRedesignedFeatureFlags,
} from '../api-mocking/mock-responses/feature-flags-mocks';

// For legacy confirmations (old UI)
const testSpecificMock = async (mockServer: Mockttp) => {
  await setupRemoteFeatureFlagsMock(
    mockServer,
    Object.assign({}, ...oldConfirmationsRemoteFeatureFlags),
  );
};

// For new confirmations UI
const testSpecificMock = async (mockServer: Mockttp) => {
  await setupRemoteFeatureFlagsMock(
    mockServer,
    Object.assign({}, ...confirmationsRedesignedFeatureFlags),
  );
};
```

### Feature Flag Override Patterns

```typescript
// Combining predefined configs with custom overrides
await setupRemoteFeatureFlagsMock(mockServer, {
  ...Object.assign({}, ...confirmationsRedesignedFeatureFlags),
  rewards: true, // Override specific flags
  carouselBanners: false,
  perpsEnabled: true, // Add new flags
});

// Environment-specific flags
await setupRemoteFeatureFlagsMock(
  mockServer,
  {
    devOnlyFeature: true,
    prodOptimization: false,
  },
  'flask',
); // Flask distribution
```

## 🎭 Complete Mocking Examples

### Basic Test with API Mocking

```typescript
import { SmokeE2E } from '../../tags';
import { withFixtures } from '../../framework/fixtures/FixtureHelper';
import FixtureBuilder from '../../framework/fixtures/FixtureBuilder';
import { Mockttp } from 'mockttp';
import { setupMockRequest } from '../../api-mocking/helpers/mockHelpers';

describe(SmokeE2E('Token Price Display'), () => {
  it('displays current token prices', async () => {
    const testSpecificMock = async (mockServer: Mockttp) => {
      await setupMockRequest(mockServer, {
        requestMethod: 'GET',
        url: 'https://price-api.metamask.io/v2/chains/1/spot-prices',
        response: {
          '0x0000000000000000000000000000000000000000': {
            // ETH
            price: 2500.5,
            currency: 'usd',
          },
        },
        responseCode: 200,
      });
    };

    await withFixtures(
      {
        fixture: new FixtureBuilder().withTokensControllerERC20().build(),
        testSpecificMock,
      },
      async () => {
        // Test implementation using mocked price API
      },
    );
  });
});
```

### Advanced Test with Multiple Mocks

```typescript
import {
  SEND_ETH_SIMULATION_MOCK,
  SIMULATION_ENABLED_NETWORKS_MOCK,
} from '../../api-mocking/mock-responses/simulations';
import { setupRemoteFeatureFlagsMock } from '../../api-mocking/helpers/remoteFeatureFlagsHelper';
import { confirmationsRedesignedFeatureFlags } from '../../api-mocking/mock-responses/feature-flags-mocks';

const testSpecificMock = async (mockServer: Mockttp) => {
  // Mock simulation API
  await setupMockRequest(mockServer, {
    requestMethod: 'GET',
    url: SIMULATION_ENABLED_NETWORKS_MOCK.urlEndpoint,
    response: SIMULATION_ENABLED_NETWORKS_MOCK.response,
    responseCode: 200,
  });

  // Mock feature flags
  await setupRemoteFeatureFlagsMock(
    mockServer,
    Object.assign({}, ...confirmationsRedesignedFeatureFlags),
  );

  // Mock transaction simulation
  const {
    urlEndpoint: simulationEndpoint,
    requestBody,
    response: simulationResponse,
    ignoreFields,
  } = SEND_ETH_SIMULATION_MOCK;

  await setupMockPostRequest(
    mockServer,
    simulationEndpoint,
    requestBody,
    simulationResponse,
    {
      statusCode: 200,
      ignoreFields,
    },
  );
};
```

### Organized Mock Setup

For complex tests with many mocks:

```typescript
const setupSwapMocks = async (mockServer: Mockttp) => {
  // Price API mocks
  await setupMockRequest(mockServer, {
    requestMethod: 'GET',
    url: 'https://price-api.metamask.io/v2/chains/1/spot-prices',
    response: {
      /* price data */
    },
    responseCode: 200,
  });

  // Swap quotes mock
  await setupMockRequest(mockServer, {
    requestMethod: 'GET',
    url: /^https:\/\/swap-api\.metamask\.io\/networks\/1\/trades/,
    response: {
      /* quote data */
    },
    responseCode: 200,
  });

  // Gas API mock
  await setupMockRequest(mockServer, {
    requestMethod: 'GET',
    url: 'https://gas-api.metamask.io/networks/1/gasPrices',
    response: {
      /* gas prices */
    },
    responseCode: 200,
  });
};

await withFixtures(
  {
    fixture: new FixtureBuilder().withGanacheNetwork().build(),
    testSpecificMock: setupSwapMocks,
  },
  async () => {
    // Complex swap test implementation
  },
);
```

## 🚨 Mocking Best Practices

### 1. Use Specific URL Patterns

```typescript
// ✅ Good - specific endpoint
url: 'https://api.metamask.io/v2/chains/1/spot-prices';

// ✅ Better - regex for dynamic parts
url: /^https:\/\/api\.metamask\.io\/v2\/chains\/\d+\/spot-prices$/;

// ❌ Avoid - too broad, may interfere with other requests
url: 'metamask.io';
```

### 2. Handle Request Bodies Properly

```typescript
// ✅ Validate important request body fields
await setupMockPostRequest(
  mockServer,
  'https://api.example.com/transactions',
  {
    to: '0x742d35cc6634c0532925a3b8d0ea4405abf5adf3',
    value: '1000000000000000000', // 1 ETH in wei
    gasLimit: '21000',
  },
  { txHash: '0x123...' },
  {
    ignoreFields: ['timestamp', 'nonce', 'gasPrice'], // Ignore dynamic fields
  },
);
```

### 3. Use Appropriate HTTP Status Codes

```typescript
// ✅ Use descriptive response codes
responseCode: 200, // OK
responseCode: 201, // Created
responseCode: 400, // Bad Request
responseCode: 404, // Not Found
responseCode: 500, // Internal Server Error
```

### 4. Organize Feature Flag Overrides

```typescript
// ✅ Use Object.assign with predefined arrays
Object.assign({}, ...confirmationsRedesignedFeatureFlags)

// ❌ Don't spread arrays directly
...confirmationsRedesignedFeatureFlags // This won't merge objects correctly
```

## 🔍 Debugging Mocks

### Request Validation and Debugging

The mock server automatically tracks and logs:

- **Matched mock requests** with request details
- **Unmocked requests** that weren't handled
- **Request/response timing** information
- **Feature flag configurations** applied

### Common Debugging Steps

1. **Check test output** for mock-related warnings
2. **Verify URL patterns** match actual requests
3. **Review request body validation** for POST requests
4. **Confirm feature flag configurations** are applied correctly
5. **Check mock precedence** (test-specific overrides defaults)

### Debug Logging

Enable debug logging to see mock activity:

```typescript
// Mock helpers automatically log when mocks are triggered
// Look for output like:
// "Mocking GET request to: https://api.example.com/data"
// "Request body validation passed for: https://api.example.com/submit"
```

## 🚨 Common Mocking Issues

### Mock Not Triggering

**Cause**: URL pattern doesn't match actual request
**Solution**: Check URL pattern specificity and regex syntax

```typescript
// ❌ Too specific - might miss query parameters
url: 'https://api.example.com/data';

// ✅ More flexible pattern
url: /^https:\/\/api\.example\.com\/data(\?.*)?$/;
```

### POST Body Validation Failing

**Cause**: Expected request body doesn't match actual request
**Solution**: Use `ignoreFields` for dynamic data

```typescript
await setupMockPostRequest(
  mockServer,
  'https://api.example.com/submit',
  { method: 'transfer', amount: '1000' },
  { success: true },
  {
    ignoreFields: ['timestamp', 'nonce', 'uuid'], // Ignore dynamic fields
  },
);
```

### Feature Flags Not Applying

**Cause**: Incorrect merging of feature flag arrays
**Solution**: Use `Object.assign({}, ...arrays)` pattern

```typescript
// ✅ Correct merging
Object.assign({}, ...confirmationsRedesignedFeatureFlags)

// ❌ Incorrect - spreads array items
{ ...confirmationsRedesignedFeatureFlags }
```

## 📚 Mocking Resources

- **Main Mocking Documentation**: [`e2e/MOCKING.md`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/MOCKING.md)
- **Mock Responses Directory**: [`e2e/api-mocking/mock-responses/`](https://github.com/MetaMask/metamask-mobile/tree/main/e2e/api-mocking/mock-responses)
- **Mock Helpers**: [`e2e/api-mocking/helpers/`](https://github.com/MetaMask/metamask-mobile/tree/main/e2e/api-mocking/helpers)
- **Feature Flag Mocks**: [`e2e/api-mocking/mock-responses/feature-flags-mocks.ts`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/api-mocking/mock-responses/feature-flags-mocks.ts)

## ✅ Mocking Checklist

Before submitting tests with custom mocks:

- [ ] Uses `testSpecificMock` parameter in `withFixtures`
- [ ] URL patterns are specific enough to avoid conflicts
- [ ] POST request body validation configured appropriately
- [ ] Uses `ignoreFields` for dynamic request data
- [ ] Appropriate HTTP status codes used
- [ ] Feature flags configured using proper merge patterns
- [ ] Mock organization follows logical grouping
- [ ] No hardcoded values that should come from constants
- [ ] Error scenarios mocked when testing error handling

The MetaMask Mobile API mocking system provides comprehensive control over network requests, enabling reliable and deterministic E2E tests. By following these patterns, you'll create tests that are both isolated and realistic.
