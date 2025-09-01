# MetaMask Mobile E2E Framework Guide

This guide covers the specific usage of MetaMask Mobile's TypeScript-based E2E testing framework. For general testing best practices, see the [General E2E Testing Guidelines](./mobile-e2e-guidelines.md).

## 🚀 Framework Overview

MetaMask Mobile uses a modern TypeScript-based E2E testing framework built on Detox, featuring:

- **Enhanced reliability** with auto-retry mechanisms
- **Type safety** with full TypeScript support
- **Configurable element state checking** (visibility, enabled, stability)
- **Comprehensive API mocking** system
- **Fixture-based test data management**

## 📋 Quick Setup

1. **Install E2E dependencies**: `yarn setup:e2e`
2. **Read setup documentation**: [`docs/readme/testing.md`](https://github.com/MetaMask/metamask-mobile/blob/main/docs/readme/testing.md)
3. **Configure environment files**: `.e2e.env` and `.js.env`
4. **Set up device**: iOS Simulator (iPhone 15 Pro) or Android Emulator (Pixel 5 API 34) - This is configurable

## 🏗️ Framework Architecture

### Modern Framework

```
e2e/framework/                    # Modern TypeScript framework
├── Assertions.ts                # Enhanced assertions with auto-retry
├── Gestures.ts                  # Robust user interactions
├── Matchers.ts                  # Type-safe element selectors
├── Utilities.ts                 # Core utilities with retry mechanisms
├── fixtures/                    # Test data management
│   ├── FixtureBuilder.ts       # Builder pattern for test fixtures
│   ├── FixtureHelper.ts        # withFixtures implementation
│   └── FixtureUtils.ts         # Utility functions
└── index.ts                     # Main entry point - import from here
```

## 📐 Essential Patterns

### MANDATORY: withFixtures Pattern

Every E2E test MUST use `withFixtures` for proper setup and cleanup:

```typescript
import { withFixtures } from '../framework/fixtures/FixtureHelper';
import FixtureBuilder from '../framework/fixtures/FixtureBuilder';
import { loginToApp } from '../viewHelper';
import { SmokeE2E } from '../tags';

describe(SmokeE2E('Feature Name'), () => {
  beforeAll(async () => {
    jest.setTimeout(150000);
  });

  it('performs expected behavior', async () => {
    await withFixtures(
      {
        fixture: new FixtureBuilder().build(),
        restartDevice: true,
      },
      async () => {
        await loginToApp();

        // Test implementation here
      },
    );
  });
});
```

### Required Framework Imports

```typescript
// ✅ ALWAYS import from framework entry point
import { Assertions, Gestures, Matchers, Utilities } from '../framework';

// ❌ NEVER import from individual files
import Assertions from '../framework/Assertions';
```

## 🎯 Framework Classes Usage

### Enhanced Assertions

```typescript
// Modern assertions with descriptions and auto-retry
await Assertions.expectElementToBeVisible(element, {
  description: 'submit button should be visible',
  timeout: 15000,
});

await Assertions.expectTextDisplayed('Success!', {
  description: 'success message should appear',
});

await Assertions.expectElementToHaveText(input, 'Expected Value', {
  description: 'input should contain entered value',
});
```

### Robust Gestures with Element State Configuration

The framework provides configurable element state checking for optimal performance and reliability:

```typescript
// Default behavior (recommended for most cases)
// checkVisibility: true, checkEnabled: true, checkStability: false
await Gestures.tap(button, {
  description: 'tap submit button',
});

// For animated elements - enable stability checking
await Gestures.tap(carouselItem, {
  checkStability: true,
  description: 'tap carousel item',
});

// For loading/disabled elements - skip checks
await Gestures.tap(loadingButton, {
  checkEnabled: false,
  description: 'tap button during loading',
});

// Text input with options
await Gestures.typeText(input, 'Hello World', {
  clearFirst: true,
  hideKeyboard: true,
  description: 'enter username',
});
```

### Type-Safe Element Matching

```typescript
// Element selectors using framework Matchers
const button = Matchers.getElementByID('send-button');
const textElement = Matchers.getElementByText('Send');
const labelElement = Matchers.getElementByLabel('Send Button');
const webElement = Matchers.getWebViewByID('dapp-webview');
```

### Retry Mechanisms and Utilities

```typescript
// High-level retry for complex interactions
await Utilities.executeWithRetry(
  async () => {
    await Gestures.tap(button, { timeout: 2000 });
    await Assertions.expectElementToBeVisible(nextScreen, { timeout: 2000 });
  },
  {
    timeout: 30000,
    description: 'tap button and verify navigation',
    elemDescription: 'Submit Button',
  },
);

// Element state checking utilities
await Utilities.checkElementReadyState(element, {
  checkVisibility: true,
  checkEnabled: true,
  checkStability: false,
});
```

## 🧪 Test Fixtures and Data Management

### FixtureBuilder Patterns

```typescript
// Basic fixture
new FixtureBuilder().build();

// With popular networks
new FixtureBuilder().withPopularNetworks().build();

// With Ganache network for local testing
new FixtureBuilder().withGanacheNetwork().build();

// With connected test dapp
new FixtureBuilder()
  .withPermissionControllerConnectedToTestDapp(buildPermissions(['0x539']))
  .build();

// With pre-configured tokens and contacts
new FixtureBuilder()
  .withAddressBookControllerContactBob()
  .withTokensControllerERC20()
  .build();
```

### Advanced withFixtures Configuration

```typescript
import {
  DappVariants,
  LocalNodeType,
  GanacheHardfork,
} from '../framework/fixtures/constants';

await withFixtures(
  {
    fixture: new FixtureBuilder().withGanacheNetwork().build(),
    restartDevice: true,

    // Configure test dapps
    dapps: [
      { dappVariant: DappVariants.MULTICHAIN },
      { dappVariant: DappVariants.TEST_DAPP },
    ],

    // Configure local blockchain nodes
    localNodeOptions: [
      {
        type: LocalNodeType.ganache,
        options: {
          hardfork: GanacheHardfork.london,
          mnemonic:
            'WORD1 WORD2 WORD3 WORD4 WORD5 WORD6 WORD7 WORD8 WORD9 WORD10 WORD11 WORD12',
        },
      },
    ],

    // Test-specific API mocks (see API Mocking Guide)
    testSpecificMock: async (mockServer) => {
      // Custom mocking logic
    },

    // Additional launch arguments
    launchArgs: {
      fixtureServerPort: 8545,
    },
  },
  async () => {
    // Test implementation
  },
);
```

## 🏛️ Page Object Integration

The framework integrates seamlessly with the Page Object Model pattern:

```typescript
import { Matchers, Gestures, Assertions } from '../../framework';
import { SendViewSelectors } from './SendView.selectors';

class SendView {
  // Getter pattern for element references
  get sendButton() {
    return Matchers.getElementByID(SendViewSelectors.SEND_BUTTON);
  }

  get amountInput() {
    return Matchers.getElementByID(SendViewSelectors.AMOUNT_INPUT);
  }

  // Action methods using framework
  async tapSendButton(): Promise<void> {
    await Gestures.tap(this.sendButton, {
      description: 'tap send button',
    });
  }

  async inputAmount(amount: string): Promise<void> {
    await Gestures.typeText(this.amountInput, amount, {
      clearFirst: true,
      description: `input amount: ${amount}`,
    });
  }

  // Verification methods
  async verifySendButtonVisible(): Promise<void> {
    await Assertions.expectElementToBeVisible(this.sendButton, {
      description: 'send button should be visible',
    });
  }

  // Complex interaction with retry
  async sendETHWithRetry(amount: string): Promise<void> {
    await Utilities.executeWithRetry(
      async () => {
        await this.inputAmount(amount);
        await this.tapSendButton();
        await Assertions.expectTextDisplayed('Transaction Sent', {
          timeout: 2000,
          description: 'transaction confirmation should appear',
        });
      },
      {
        timeout: 30000,
        description: 'complete send ETH transaction',
      },
    );
  }
}

export default new SendView();
```

## ❌ Framework Anti-Patterns

### Prohibited Patterns

```typescript
// ❌ NEVER: Use TestHelpers.delay()
await TestHelpers.delay(5000);

// ❌ NEVER: Use deprecated methods (marked with @deprecated)
await Assertions.checkIfVisible(element);
await Gestures.tapAndLongPress(element);

// ❌ NEVER: Import from individual framework files
import Assertions from '../framework/Assertions';

// ❌ NEVER: Missing descriptions in framework calls
await Gestures.tap(button);
await Assertions.expectElementToBeVisible(element);

// ❌ NEVER: Manual retry loops (use framework utilities)
let attempts = 0;
while (attempts < 5) {
  try {
    await Gestures.tap(button);
    break;
  } catch {
    attempts++;
  }
}
```

### Correct Framework Usage

```typescript
// ✅ Use proper waiting with framework assertions
await Assertions.expectElementToBeVisible(element, {
  description: 'element should be visible',
});

// ✅ Use modern framework methods with descriptions
await Gestures.tap(button, { description: 'tap submit button' });

// ✅ Use framework retry mechanisms
await Utilities.executeWithRetry(
  async () => {
    /* operation */
  },
  { timeout: 30000, description: 'retry operation' },
);
```

## 🚨 Framework Troubleshooting

### Common Framework Issues

#### "Element not enabled" Errors

```typescript
// Solution: Skip enabled check for temporarily disabled elements
await Gestures.tap(loadingButton, {
  checkEnabled: false,
  description: 'tap button during loading state',
});
```

#### "Element moving/animating" Errors

```typescript
// Solution: Enable stability checking for animated elements
await Gestures.tap(animatedButton, {
  checkStability: true,
  description: 'tap animated carousel button',
});
```

#### Framework Migration Issues

```typescript
// ❌ Old deprecated pattern
await Assertions.checkIfVisible(element, 15000);

// ✅ New framework pattern
await Assertions.expectElementToBeVisible(element, {
  timeout: 15000,
  description: 'element should be visible',
});
```

## 🔄 Migration from Legacy Framework

### Migration Status

Current migration phases from [`e2e/framework/README.md`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/framework/README.md):

- ✅ Phase 0: TypeScript framework foundation
- ✅ Phase 1: ESLint for E2E tests
- ⏳ Phase 2: Legacy framework replacement
- ⏳ Phase 3: Gradual test migration

### For New Tests

- Use TypeScript framework exclusively
- Import from `e2e/framework/index.ts`
- Follow all patterns in this guide
- Use `withFixtures` pattern

### For Existing Tests

- Gradually migrate to TypeScript framework
- Replace deprecated methods (check `@deprecated` tags)
- Update imports to use framework entry point
- Add `description` parameters to all framework calls

### Common Migration Patterns

| Legacy Pattern                                 | Modern Framework Equivalent                                               |
| ---------------------------------------------- | ------------------------------------------------------------------------- |
| `TestHelpers.delay(5000)`                      | `Assertions.expectElementToBeVisible(element, {timeout: 5000})`           |
| `checkIfVisible(element, 15000)`               | `expectElementToBeVisible(element, {timeout: 15000, description: '...'})` |
| `waitFor(element).toBeVisible()`               | `expectElementToBeVisible(element, {description: '...'})`                 |
| `tapAndLongPress(element)`                     | `longPress(element, {description: '...'})`                                |
| `clearField(element); typeText(element, text)` | `typeText(element, text, {clearFirst: true, description: '...'})`         |

## 📚 Framework Resources

- **Framework Documentation**: [`e2e/framework/README.md`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/framework/README.md)
- **Setup Guide**: [`docs/readme/testing.md`](https://github.com/MetaMask/metamask-mobile/blob/main/docs/readme/testing.md)
- **API Mocking Guide**: [API Mocking Documentation](./e2e-api-mocking-guide.md)
- **Testing Guidelines**: [`e2e/.cursor/rules/e2e-testing-guidelines.mdc`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/.cursor/rules/e2e-testing-guidelines.mdc)
- **Fixtures Documentation**: [`e2e/framework/fixtures/README.md`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/framework/fixtures/README.md)

## ✅ Framework Checklist

Before using the framework in tests, ensure:

- [ ] Uses `withFixtures` pattern for all test setup
- [ ] Imports from `e2e/framework/index.ts` (not individual files)
- [ ] No usage of `TestHelpers.delay()` or deprecated methods
- [ ] All framework calls include descriptive `description` parameters
- [ ] Element state configuration used appropriately (visibility, enabled, stability)
- [ ] FixtureBuilder used for test data setup
- [ ] Framework retry mechanisms used instead of manual loops
- [ ] TypeScript types used for better development experience

The MetaMask Mobile E2E framework provides a robust, reliable foundation for writing maintainable end-to-end tests. By following these patterns and avoiding anti-patterns, you'll create tests that are both resilient and easy to understand.
