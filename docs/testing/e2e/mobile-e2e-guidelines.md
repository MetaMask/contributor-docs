# MetaMask Mobile E2E Testing Guidelines

**Crafting Tests Like a Masterpiece**

In the realm of software testing, writing test code is akin to composing a compelling narrative. Just as a well-constructed story draws readers in with clarity and coherence, high-quality test code should engage developers with its readability and clarity.

## 🧪 Test Classification and Philosophy

### Are These E2E Tests or System Tests?

MetaMask Mobile's "E2E" tests are technically **system tests** with controlled dependencies:

- **System Under Test**: Complete MetaMask Mobile app running on real devices/simulators
- **External Dependencies**: Mocked APIs, controlled blockchain networks (Ganache), test dapps
- **User Interface**: Real UI interactions through Detox framework
- **Data Flow**: Complete user journeys from UI through business logic to mocked external services

**Why We Call Them E2E**: The terminology reflects the **user perspective** - tests cover complete user workflows from end to end, even though external dependencies are controlled for reliability.

### Benefits of This Approach

- **Deterministic**: Mocked APIs ensure consistent test results
- **Fast**: No network latency or external service dependencies
- **Isolated**: Tests don't affect real services or depend on external state
- **Comprehensive**: Full app stack tested with realistic user interactions
- **Maintainable**: Controlled environment reduces flakiness

To ensure that our tests convey their purpose effectively, let's delve into some foundational principles:

## 🗺️ Locator Strategy

The default strategy for locating elements is using Test IDs. However, in complex scenarios, employ Text or Label-based locators.

### Locating Elements byID

To add Test IDs to a component, consult Detox's [guidelines](https://wix.github.io/Detox/docs/guide/test-id).
Store Test IDs in separate `'{PAGE}.selectors.js` files, corresponding to each page object. For example, the page object `PrivacyAndSecurity` should have a testIDs file called `PrivacyAndSecurity.selectors.js`

### Locating Elements byLabel

In some instances on Android, elements are not located by TestIDs. The alternative is to locate elements byLabel.

✅ Good: Utilize the `byLabel()` element locator on Android when Test IDs aren't detected correctly.

```javascript
device.getPlatform() === 'android'
  ? Matchers.getElementByLabel('ELEMENT-ID')
  : Matchers.getElementByID('ELEMENT-ID');
```

❌ Bad: Attempting to create android specific and ios specific locators:

```javascript
device.getPlatform() === 'android'
  ? Matchers.getElementByLabel('ANDROID-ELEMENT-ID')
  : Matchers.getElementByID('iOS-ELEMENT-ID');
```

### Locating Elements byText

✅ Good: When locating elements by text, retrieve the corresponding text string from the `en.json` file in the [`locales/languages`](https://github.com/MetaMask/metamask-mobile/tree/main/locales/languages) folder. For instance, if you need to interact with `Show Hex Data`, access it as follows:

```javascript
import en from '../../locales/languages/en.json';
en.app_settings.show_hex_data;
```

❌ Bad: Hardcoding text strings without retrieving them from the language file, like this:

```javascript
const elementText = 'Show Hex Data'; // Hardcoded text
```

## 📌 Naming Conventions

Descriptive and clear names are the cornerstone of maintainable code. Vague, cryptic names leave readers puzzled.

### Variables/Methods

✅ Good: Name variables and methods with utmost clarity.

```javascript
tapCreateWalletButton() {
  // Implementation
}
```

❌ Bad: Using abbreviated or unclear names for variables and methods, like this:

```javascript
tapNTB() {
  // What does NTB mean?
}
```

or

```javascript
tapBtn() {
  // Which button?
}
```

### Test IDs

When crafting Selector objects, it is crucial to name the object in a way that identifies the specific page where the elements are. The Selector object can be of type ID or string, and it is essential to explicitly specify this distinction. For instance, when creating selectors for the AddCustomToken view, planning to utilize both testIDs and strings as locator strategies, two separate Selector objects should be created.

When naming selector objects follow this pattern:
`{ScreenName}Selectors{Type}`. In this case `{TYPE}` means the selector type, i.e. IDs or Strings.

✅ Good:

```javascript
export const AddCustomTokenViewSelectorsIDs = {};
export const AddCustomTokenViewSelectorsText = {};
```

❌ Bad: Avoid generic names.

```javascript
export const customTokens = {};
```

For selector keys, use uppercase letters to define them. Follow the format ELEMENT_NAME.

For instance:

✅ Good:

```javascript
export const AddCustomTokenViewSelectorsIDs = {
  TOKEN_SYMBOL: '',
};
```

❌ Bad:

```javascript
export const AddCustomTokenViewSelectorsIDs = {
  tokenSymbol: '',
};
```

For selector values, craft specific, lowercase strings using the format `{screen}-{elementname}`.

For example:

✅ Good:

```javascript
export const AddCustomTokenViewSelectorsIDs = {
  TOKEN_SYMBOL: 'token-screen-symbol',
};
```

❌ Bad: Avoid employing generic, ambiguous strings

```javascript
export const AddCustomTokenViewSelectorsIDs = {
  TOKEN_SYMBOL: 'symbol',
};
```

🚨 A crucial point to note is that if you are creating selector string objects, it is recommended to utilize strings from a localization file.

✅ Good:

```javascript
import locales from '../../locales/languages/en.json';

export const AddCustomTokenViewSelectorsText = {
  IMPORT_BUTTON: locales.add_asset.tokens.add_token,
};
```

❌ Bad:

```javascript
export const AddCustomTokenViewSelectorsText = {
  IMPORT_BUTTON: 'add-token',
};
```

In summary: Create Selector objects for locating elements. If your strategy involves locating by ID or by text, ensure that the Selector object name reflects that strategy. The object should contain key-value pairs, where keys represent different UI elements and values are unique identifiers.

```javascript
export const AddCustomTokenViewSelectorsIDs = {
  CANCEL_BUTTON: 'add-custom-asset-cancel-button',
  CONFIRM_BUTTON: 'add-custom-asset-confirm-button',
};
```

Or in terms of creating a selector string object:

```javascript
export const AddCustomTokenViewSelectorsText = {
  TOKEN_SYMBOL: messages.token.token_symbol,
  IMPORT_BUTTON: messages.add_asset.tokens.add_token,
};
```

❌ Bad:

```javascript
const DELETE_WALLET_INPUT_BOX_ID = 'delete-wallet-input-box';
```

## ✨ Mobile Custom Framework

To ensure consistency and reliability in our test scripts, we use MetaMask Mobile's E2E framework. For detailed framework usage, see the [E2E Framework Guide](./mobile-e2e-framework-guide.md).

✅ Good: Utilize framework methods with descriptions:

```typescript
import { Assertions, Gestures, Matchers } from '../framework';

await Assertions.expectElementToBeVisible(
  SecurityAndPrivacy.metaMetricsToggle,
  {
    description: 'MetaMetrics toggle should be visible',
  },
);

await Gestures.tap(button, {
  description: 'tap create wallet button',
});
```

❌ Bad: Using legacy patterns or missing descriptions:

```javascript
await expect(element(by.id('login-button'))).toHaveText('login-text');
await element(by.id('button')).tap(); // No description
```

_NOTE: Generally speaking, you don't need to put an assertion before a test action. You can minimize the amount of assertions by using a test action as an implied assertion on the same element._

## 🛠️ Method Design

✅ Good: Embrace the "Don't Repeat Yourself" (DRY) principle. Reuse existing page-object actions to prevent code duplication and maintain code integrity. Ensure each method has a singular purpose.

```typescript
async tapCreateWalletButton(): Promise<void> {
  await Gestures.tap(this.createWalletButton, {
    description: 'tap create wallet button',
  });
}

async verifyWalletCreated(): Promise<void> {
  await Assertions.expectTextDisplayed('Wallet Created Successfully', {
    description: 'wallet creation confirmation should appear',
  });
}
```

❌ Bad: Cluttering methods with multiple responsibilities and duplicating code:

```javascript
tapNoThanksButton() {
  await OnboardingWizard.isVisible();
  await TestHelpers.waitAndTap(NO_THANKS_BUTTON_ID);
  await TestHelpers.isNoThanksButtonNotVisible();
  // Too many responsibilities in one method
}
```

## 📝 Comments and Documentation

✅ Good: Use JSDoc syntax for adding documentation to clarify complex logic or provide context. Excessive comments can be counterproductive and may signal a need for code simplification.

```typescript
/**
 * Get element by web ID for dApp interactions.
 *
 * @param {string} webID - The web ID of the element to locate within the webview
 * @return {Promise} Resolves to the located web element
 */
async getElementByWebID(webID: string): Promise<WebElement> {
  return Matchers.getWebViewByID(webID);
}

/**
 * Completes the full send ETH workflow including validation and confirmation.
 * Handles network switching if required and validates gas estimation.
 *
 * @param {string} recipient - The recipient ETH address (must be valid)
 * @param {string} amount - The amount to send in ETH (e.g., "0.1")
 * @param {boolean} maxGas - Whether to use maximum gas limit
 */
async sendETHTransaction(recipient: string, amount: string, maxGas = false): Promise<void> {
  // Complex workflow implementation
}
```

❌ Bad: Over commenting simple and self-explanatory code:

```javascript
// This function taps the button
tapButton() {
  // Tap the button
  await Gestures.tap(this.button);
}
```

## 🏛️ Creating Page Objects

Page objects serve as the building blocks of our test suites, providing a clear and organized representation of the elements and interactions within our application.

When creating page objects, follow these principles to ensure clarity, maintainability, and reusability.

### 🔁 Consistency in Structure

Establishing a well-defined structure for your page objects is crucial as it facilitates better understanding and code navigation for fellow engineers.

✅ Good: Define a clear structure for your page objects, organizing elements and actions in a logical manner. This makes it easy to reuse elements and actions across multiple tests. For example:

#### Page object

```typescript
import { Matchers, Gestures, Assertions } from '../../framework';
import { SettingsViewSelectors } from './SettingsView.selectors';

class SettingsView {
  // Element getters
  get networksButton() {
    return Matchers.getElementByID(SettingsViewSelectors.NETWORKS_BUTTON);
  }

  // Action methods
  async tapNetworksButton(): Promise<void> {
    await Gestures.tap(this.networksButton, {
      description: 'tap networks button',
    });
  }

  // Verification methods
  async verifySettingsPageVisible(): Promise<void> {
    await Assertions.expectElementToBeVisible(this.networksButton, {
      description: 'settings page should be visible',
    });
  }
}

export default new SettingsView();
```

#### Spec file:

```typescript
import { withFixtures } from '../framework/fixtures/FixtureHelper';
import FixtureBuilder from '../framework/fixtures/FixtureBuilder';
import SettingsView from '../pages/Settings/SettingsView';

it('navigates to networks settings', async () => {
  await withFixtures(
    {
      fixture: new FixtureBuilder().build(),
      restartDevice: true,
    },
    async () => {
      await SettingsView.verifySettingsPageVisible();
      await SettingsView.tapNetworksButton();
    },
  );
});
```

❌ Bad: Tests Do Not Adhere to Page Objects. When tests do not adhere to page objects, it leads to code that is difficult to maintain, prone to duplication, and lacks clarity in its structure.

```javascript
it('create a new wallet', async () => {
  // Direct element interaction without page objects
  await TestHelpers.waitAndTap('start-exploring-button');
  await TestHelpers.checkIfVisible('metaMetrics-OptIn');
});
```

### 🏷️ Meaningful Naming

✅ Good: Choose descriptive names for page object properties and methods that accurately convey their purpose:

```typescript
class WalletView {
  get sendButton() {
    /* ... */
  }
  get receiveButton() {
    /* ... */
  }

  async tapSendButton(): Promise<void> {
    /* ... */
  }
  async initiateReceiveFlow(): Promise<void> {
    /* ... */
  }
}
```

❌ Bad: Unclear or ambiguous names that make it difficult to understand the purpose:

```javascript
class Screen2 {
  get btn1() {
    /* ... */
  }
  get thing() {
    /* ... */
  }
}
```

### 📦 Encapsulation of Interactions

By encapsulating interaction logic within methods, test code does not need to concern itself with the specifics of how interactions are performed. This allows for easier modification of interaction behavior without impacting test scripts.

✅ Good: Encapsulate interactions with page elements within methods, abstracting away implementation details:

```typescript
class SettingsView {
  async navigateToNetworks(): Promise<void> {
    await Gestures.tap(this.networksButton, {
      description: 'navigate to networks settings',
    });
    await Assertions.expectElementToBeVisible(NetworkView.networkContainer, {
      description: 'networks page should load',
    });
  }
}
```

❌ Bad: Exposing implementation details and directly interacting with elements in test code:

```javascript
// Test code with direct element interaction
await TestHelpers.waitAndTap('networks-button');
await TestHelpers.checkIfVisible('network-container');
```

### ⚙️ Utilization of Modern Framework

Using the modern TypeScript framework is essential for consistent and reliable interactions across tests. The framework centralizes interaction logic and removes low-level details, improving readability and enhancing test scalability.

✅ Good: Leverage the modern framework for all page object interactions:

```typescript
import { Matchers, Gestures, Assertions } from '../../framework';
import { NetworksViewSelectors } from './NetworksView.selectors';

class NetworksView {
  get networksButton() {
    return Matchers.getElementByID(NetworksViewSelectors.NETWORKS_BUTTON);
  }

  async tapNetworksButton(): Promise<void> {
    await Gestures.tap(this.networksButton, {
      description: 'tap networks button',
    });
  }

  async verifyNetworksVisible(): Promise<void> {
    await Assertions.expectElementToBeVisible(this.networksButton, {
      description: 'networks button should be visible',
    });
  }
}
```

_NOTE:_ The modern framework components (`Matchers`, `Gestures`, `Assertions`) are fundamental to our test reliability and maintainability.

- **Matchers**: Handles element identification with type safety and platform compatibility
- **Gestures**: Manages user interactions with configurable state checking and auto-retry
- **Assertions**: Provides enhanced verification with descriptive error messages and retry logic

❌ Bad: Using legacy patterns or direct Detox calls without the framework:

```javascript
await element(by.id('networks-button')).tap();
await waitFor(element(by.id('network-container'))).toBeVisible();
```

### 🗃️ Using Getters for Element Storage

In our page objects, we utilize getters for element storage instead of defining selectors within the constructor. This deliberate choice stems from the behavior of getters, which are evaluated when you access the property, not when you create the object.

✅ Good: By encapsulating element selectors within getters, we ensure that elements are requested immediately before any action is taken on them. This approach promotes a more dynamic and responsive interaction with the page elements, enhancing the reliability and robustness of our tests.

```typescript
import { NetworksViewSelectors } from '../../selectors/Settings/NetworksView.selectors';
import { Matchers } from '../../framework';

class NetworksView {
  get networksButton() {
    return Matchers.getElementByID(NetworksViewSelectors.NETWORKS_BUTTON);
  }

  get addNetworkButton() {
    return device.getPlatform() === 'ios'
      ? Matchers.getElementByID(NetworksViewSelectors.ADD_NETWORK_BUTTON)
      : Matchers.getElementByLabel(NetworksViewSelectors.ADD_NETWORK_BUTTON);
  }

  async tapNetworksButton(): Promise<void> {
    await Gestures.tap(this.networksButton, {
      description: 'tap networks button',
    });
  }
}
```

❌ Bad: Defining element selectors as constants can lead to potential issues with element staleness and unexpected behavior:

```javascript
const CONFIRM_BUTTON_ID = 'contract-name-confirm-button';

export default class ContractNickNameView {
  static async tapConfirmButton() {
    await TestHelpers.waitAndTap(CONFIRM_BUTTON_ID);
  }
}
```

## 📖 A Comprehensive Guide on Implementing Page Objects

This guide provides a comprehensive overview of implementing the concepts discussed earlier to create a page object for use in a test scenario. By following this guide, you will gain a deeper insight into our approach to writing tests, enabling you to produce test code that is both maintainable and reusable.

### Step 1: Define Selector Objects for NetworksView

Start by creating selector objects. These objects store identifiers for UI elements, making your tests easier to read and maintain. We'll use two types of selectors: IDs and Text. Let's call this file: [`NetworksView.selectors.ts`](https://github.com/MetaMask/metamask-mobile/tree/main/e2e/selectors/Settings)

```typescript
import enContent from '../../../locales/languages/en.json';

export const NetworksViewSelectors = {
  RPC_CONTAINER: 'new-rpc-screen',
  ADD_NETWORKS_BUTTON: 'add-network-button',
  NETWORK_NAME_INPUT: 'input-network-name',
  BLOCK_EXPLORER_INPUT: 'block-explorer',
  RPC_URL_INPUT: 'input-rpc-url',
  CHAIN_INPUT: 'input-chain-id',
  NETWORKS_SYMBOL_INPUT: 'input-network-symbol',
  RPC_WARNING_BANNER: 'rpc-url-warning',
  NETWORK_CONTAINER: 'networks-screen',
  CUSTOM_NETWORK_LIST: 'custom-networks-list',
};

export const NetworkViewSelectorsText = {
  BLOCK_EXPLORER: enContent.app_settings.network_block_explorer_label,
  REMOVE_NETWORK: enContent.app_settings.remove_network,
  CUSTOM_NETWORK_TAB: enContent.app_settings.custom_network_name,
  POPULAR_NETWORK_TAB: enContent.app_settings.popular,
};
```

### Step 2: Create the Networks Page Object

The Networks Page Object encapsulates interactions with the NetworksView using the modern framework. It uses the selectors defined in _Step 1_ and framework utilities for enhanced reliability.

```typescript
import {
  NetworksViewSelectors,
  NetworkViewSelectorsText,
} from '../../selectors/Settings/NetworksView.selectors';
import { Matchers, Gestures, Assertions } from '../../framework';

class NetworkView {
  get networkContainer() {
    return Matchers.getElementByID(NetworksViewSelectors.NETWORK_CONTAINER);
  }

  get addNetworkButton() {
    return device.getPlatform() === 'ios'
      ? Matchers.getElementByID(NetworksViewSelectors.ADD_NETWORKS_BUTTON)
      : Matchers.getElementByLabel(NetworksViewSelectors.ADD_NETWORKS_BUTTON);
  }

  get customNetworkTab() {
    return Matchers.getElementByText(
      NetworkViewSelectorsText.CUSTOM_NETWORK_TAB,
    );
  }

  get rpcWarningBanner() {
    return Matchers.getElementByID(NetworksViewSelectors.RPC_WARNING_BANNER);
  }

  get rpcURLInput() {
    return Matchers.getElementByID(NetworksViewSelectors.RPC_URL_INPUT);
  }

  get networkNameInput() {
    return Matchers.getElementByID(NetworksViewSelectors.NETWORK_NAME_INPUT);
  }

  get chainIDInput() {
    return Matchers.getElementByID(NetworksViewSelectors.CHAIN_INPUT);
  }

  get networkSymbolInput() {
    return Matchers.getElementByID(NetworksViewSelectors.NETWORKS_SYMBOL_INPUT);
  }

  async typeInNetworkName(networkName: string): Promise<void> {
    await Gestures.typeText(this.networkNameInput, networkName, {
      clearFirst: true,
      hideKeyboard: true,
      description: `enter network name: ${networkName}`,
    });
  }

  async typeInRpcUrl(rpcUrl: string): Promise<void> {
    await Gestures.typeText(this.rpcURLInput, rpcUrl, {
      clearFirst: true,
      hideKeyboard: true,
      description: `enter RPC URL: ${rpcUrl}`,
    });
  }

  async clearRpcInputBox(): Promise<void> {
    await Gestures.typeText(this.rpcURLInput, '', {
      clearFirst: true,
      description: 'clear RPC URL input',
    });
  }

  async tapAddNetworkButton(): Promise<void> {
    await Gestures.tap(this.addNetworkButton, {
      description: 'tap add network button',
    });
  }

  async switchToCustomNetworks(): Promise<void> {
    await Gestures.tap(this.customNetworkTab, {
      description: 'switch to custom networks tab',
    });
  }

  async typeInChainId(chainID: string): Promise<void> {
    await Gestures.typeText(this.chainIDInput, chainID, {
      clearFirst: true,
      hideKeyboard: true,
      description: `enter chain ID: ${chainID}`,
    });
  }

  async typeInNetworkSymbol(networkSymbol: string): Promise<void> {
    await Gestures.typeText(this.networkSymbolInput, networkSymbol, {
      clearFirst: true,
      hideKeyboard: true,
      description: `enter network symbol: ${networkSymbol}`,
    });
  }

  async verifyRpcWarningVisible(): Promise<void> {
    await Assertions.expectElementToBeVisible(this.rpcWarningBanner, {
      description: 'RPC warning banner should be visible',
    });
  }
}

export default new NetworkView();
```

### Step 3: Page Objects

With the page object in place and using the modern framework with `withFixtures`, writing test specifications becomes more straightforward and reliable.

#### Example Test Case: Adding a Network

```typescript
import { withFixtures } from '../../framework/fixtures/FixtureHelper';
import FixtureBuilder from '../../framework/fixtures/FixtureBuilder';
import { loginToApp } from '../../viewHelper';
import NetworkView from '../../pages/Settings/NetworksView';
import { SmokeNetworks } from '../../tags';

describe(SmokeNetworks('Network Management'), () => {
  beforeAll(async () => {
    jest.setTimeout(150000);
  });

  it('adds custom network with validation', async () => {
    await withFixtures(
      {
        fixture: new FixtureBuilder().build(),
        restartDevice: true,
      },
      async () => {
        await loginToApp();

        // Navigate and start adding network
        await NetworkView.tapAddNetworkButton();
        await NetworkView.switchToCustomNetworks();

        // Enter network details with validation
        await NetworkView.typeInNetworkName('Gnosis Test Network');

        // Test validation with incorrect RPC URL
        await NetworkView.typeInRpcUrl('invalid-url');
        await NetworkView.verifyRpcWarningVisible();

        // Clear and enter correct RPC URL
        await NetworkView.clearRpcInputBox();
        await NetworkView.typeInRpcUrl('https://rpc.gnosischain.com');
        await NetworkView.typeInChainId('100');
        await NetworkView.typeInNetworkSymbol('GNO');
      },
    );
  });
});
```

## 🧪 Test Design Philosophy

### Test Atomicity and Coupling

#### When to Isolate Tests:

- Testing specific functionality of a single component or feature
- When you need to pinpoint exact failure causes
- For basic unit-level behaviors

#### When to Combine Tests:

- For multi-step user flows that represent real user behavior
- When testing how different parts of the application work together
- When the setup for multiple tests is time-consuming and identical

#### Guidelines:

- Each test should run with a dedicated app instance and controlled environment
- Use `withFixtures` to create test prerequisites and clean up afterward
- Control application state programmatically rather than through UI interactions
- Consider the "fail-fast" philosophy - if an initial step fails, subsequent steps may not need to run

### Controlling State

#### Best Practices:

- Control application state through fixtures rather than UI interactions
- Use `FixtureBuilder` to set up test prerequisites instead of UI steps
- Minimize UI interactions to reduce potential breaking points
- Improve test stability by reducing timing and synchronization issues

#### Example:

```typescript
// ✅ Good: Use fixture to set up prerequisites
const fixture = new FixtureBuilder()
  .withAddressBookControllerContactBob()
  .withTokensControllerERC20()
  .build();

await withFixtures({ fixture }, async () => {
  await loginToApp();
  // Test only the essential steps:
  // - Navigate to send flow
  // - Select contact from address book
  // - Send TST token
  // - Verify transaction
});

// ❌ Bad: Building all state through UI
await withFixtures({ fixture: new FixtureBuilder().build() }, async () => {
  await loginToApp();
  // All these steps add complexity and potential failure points:
  // - Navigate to contacts
  // - Add contact manually
  // - Navigate to dapp
  // - Connect to dapp
  // - Deploy token contract
  // - Add token to wallet
  // - Navigate to send
  // - Send token
});
```

## 🎯 Test Quality Principles

### Reliability

- Tests should consistently produce the same results
- Use controlled environments and mocked external dependencies
- Implement proper retry mechanisms through the framework
- Handle expected failures gracefully

### Maintainability

- Follow Page Object Model pattern consistently
- Use descriptive naming throughout
- Keep tests focused on specific behaviors
- Minimize code duplication through reusable page objects

### Readability

- Write tests that tell a story of user behavior
- Use meaningful descriptions in all framework calls
- Structure tests logically with clear arrange-act-assert patterns
- Document complex test scenarios with comments when necessary

### Speed

- Use `withFixtures` for efficient test setup
- Minimize unnecessary UI interactions
- Leverage framework optimizations (like `checkStability: false` by default)
- Group related assertions to reduce redundant operations

## 📚 Additional Resources

- **Framework Usage Guide**: [E2E Framework Guide](./mobile-e2e-framework-guide.md)
- **API Mocking Guide**: [E2E API Mocking Guide](./mobile-e2e-api-mocking-guide.md)
- **Setup Documentation**: [`docs/readme/testing.md`](https://github.com/MetaMask/metamask-mobile/blob/main/docs/readme/testing.md)
- **Framework Documentation**: [`e2e/framework/README.md`](https://github.com/MetaMask/metamask-mobile/blob/main/e2e/framework/README.md)

By following these timeless testing principles alongside the modern MetaMask Mobile framework, you'll create test suites that are not only functional but also serve as living documentation of how the application should behave from a user's perspective.
