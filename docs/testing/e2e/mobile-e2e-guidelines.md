# MetaMask Mobile E2E Test Guidelines

Crafting Tests Like a Masterpiece

In the realm of software testing, writing test code is akin to composing a compelling narrative. Just as a well-constructed story draws readers in with clarity and coherence, high-quality test code should engage developers with its readability and clarity. To ensure that our tests convey their purpose effectively, let's delve into some foundational principles:

## 🗺 Locator Strategy

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

✅ Good: When locating elements by text, retrieve the corresponding text string from the `en.json` file in the `locales/languages` folder. For instance, if you need to interact with `Show Hex Data`, access it as follows:

```javascript
import en from '../../locales/languages/en.json';
en.app_settings.show_hex_data;
```

❌ Bad: Hardcoding text strings without retrieving them from the language file, like this:

```javascript
const elementText = 'Show Hex Data'; // Hardcoded text
```

## 📌 Naming Convention

Descriptive and clear names are the cornerstone of maintainable code. Vague, cryptic names leave readers puzzled.

### Variables/Methods

✅ Good: Name variables and methods with utmost clarity.

```javascript
tapCreateWalletButton() {
}
```

❌ Bad: Using abbreviated or unclear names for variables and methods, like this:

```javascript
tapNTB() {
}
```

or

```javascript
tapBtn() {
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

## Assertions

To ensure consistency and avoid redundancy in our test scripts, we've implemented an Assertion class. Whenever assertions are needed within tests, it's preferred to utilize methods provided by this class.

✅ Good: Utilize assertion methods from the Assertion class to perform assertions in tests. For instance:

```javascript
await Assertions.checkIfToggleIsOn(SecurityAndPrivacy.metaMetricsToggle);
```

❌ Bad: Refrain from using built-in assertion methods directly within tests, as shown below:

```javascript
await expect(element(by.id(Login - button))).toHaveText(login - text);
```

_NOTE: Generally speaking, you don’t need to put an assertion before a test action. You can minimize the amount of assertions by using a test action as an implied assertion on the same element._

## 🛠 Method Design

✅ Good: Embrace the "Don't Repeat Yourself" (DRY) principle. Reuse existing page-object actions to prevent code duplication and maintain code integrity. Ensure each method has a singular purpose.

❌ Bad: Cluttering methods with multiple tasks and duplicating code, like this:

```javascript

tapNoThanksButton() {
  await OnboardingWizard.isVisible();
  await TestHelpers.waitAndTap(NO_THANKS_BUTTON_ID);
  await TestHelpers.isNoThanksButtonNotVisible();
}
```

## 📝 Comments

✅ Good: Use JSDoc syntax for adding documentation to clarify complex logic or provide context. Excessive comments can be counterproductive and may signal a need for code simplification.

```javascript

/**
  * Get element by web ID.
  *
  * @param {string} webID - The web ID of the element to locate
  * @return {Promise} Resolves to the located element
  */
 async getElementByWebID(webID) {
   return web.element(by.web.id(webID));
 }
```

❌ Bad: Over Commenting simple and self-explanatory code, like this:

```javascript

// This function taps the button
tapButton() {
  // ...
```

## Creating Page Objects:

Page objects serve as the building blocks of our test suites, providing a clear and organized representation of the elements and interactions within our application.

When creating page objects, follow these principles to ensure clarity, maintainability, and reusability.

### 🔁 Consistency in Structure

Establishing a well-defined structure for your page objects is crucial as it facilitates better understanding and code navigation for fellow engineers.

✅ Good: Define a clear structure for your page objects, organizing elements and actions in a logical manner. This makes it easy to reuse elements and actions across multiple tests. For example:

##### Page object

```javascript
class SettingsPage {
  // Element selectors
  get networksButton() {
    /*...*/
  }

  // Actions
  async tapNetworksButton() {
    /*...*/
  }
}
```

##### Spec file:

```javascript
it('tap networks button', async () => {
  await SettingsView.tapNetworks();
});
```

❌ Bad: Tests Do Not Adhere to Page Objects. When tests do not adhere to page objects, it leads to code that is difficult to maintain, prone to duplication, and lacks clarity in its structure.

```javascript
it('create a new wallet', async () => {
  // Check that Start Exploring CTA is visible & tap it
  await TestHelpers.waitAndTap('start-exploring-button');
  // Check that we are on the metametrics optIn screen
  await TestHelpers.checkIfVisible('metaMetrics-OptIn');
});
```

### 🏷️ Meaningful Naming

Good:

✅ Choose descriptive names for page object properties and methods that accurately convey their purpose. For example:

```javascript
class SettingsPage {}
```

❌ Bad: Unclear or ambiguous names that make it difficult to understand the purpose of the page object:

```javascript
class Screen2 {}
```

### 📦 Encapsulation of Interactions

By encapsulating interaction logic within methods, test code does not need to concern itself with the specifics of how interactions are performed. This allows for easier modification of interaction behavior without impacting test scripts.

✅ Good: Encapsulate interactions with page elements within methods, abstracting away implementation details. For example:

```javascript
class SettingsPage {
  async tapNetworksButton() {
    /*...*/
  }
}
```

❌ Bad: Exposing implementation details and directly interacting with elements in test code:

```javascript
// Test code
await TestHelpers.waitAndTap('start-exploring-button');
```

### ⚙️ Utilization of Utility Functions

Using utility functions is a reliable way to interact with page elements consistently across various tests. By centralizing the interaction logic and removing low-level details, utility functions improve readability and enhance the scalability of tests.

✅ Good: Leverage utility functions to interact with page elements consistently across tests:

```javascript
import Matchers from '../../utils/Matchers';
import Gestures from '../../utils/Gestures';

class SettingsPage {
  get networksButton() {
    return Matchers.getElementByID('ELEMENT-STRING');
  }

  async tapNetworksButton() {
    await Gestures.waitAndTap(this.networksButton);
  }
}
```

_NOTE:_ Matchers and Gestures are fundamental components of our test actions.

Matchers Utility Class: Handles element identification and matching logic, ensuring consistency in element location across tests. This promotes consistency and reduces code duplication.

Gestures Utility Class: Manages user interactions with page elements, such as tapping, swiping, or scrolling. It enhances test readability by providing descriptive methods for common user interactions.

❌ Bad: Implementing interactions directly in test code without using utility functions. For example:

```javascript
await element(by.id('ELEMENT-STRING')).tap();
```

### 🗃️ Using Getters for Element Storage

In our page objects, we utilize getters for element storage instead of defining selectors within the constructor. This deliberate choice stems from the behavior of getters, which are evaluated when you access the property, not when you create the object.

✅ Good: By encapsulating element selectors within getters, we ensure that elements are requested immediately before any action is taken on them. This approach promotes a more dynamic and responsive interaction with the page elements, enhancing the reliability and robustness of our tests.

For example:

```javascript
import { NetworksViewSelectorsIDs } from '../../selectors/Settings/NetworksView.selectors';

class SettingsPage {
  get networksButton() {
    return Matchers.getElementByID(SettingsViewSelectorsIDs.NETWORKS);
  }

  async tapNetworksButton() {
    await Gestures.waitAndTap(this.networksButton);
  }
}
```

❌ Bad: Defining element selectors as constants and then interacting with them within test actions can lead to potential issues with element staleness and unexpected behavior.

```javascript
const CONFIRM_BUTTON_ID = 'contract-name-confirm-button';

export default class ContractNickNameView {
  static async tapConfirmButton() {
    await TestHelpers.waitAndTap(CONFIRM_BUTTON_ID);
  }
}
```

## A Comprehensive Guide on Implementing Page Objects for Test Scenarios

This guide provides a comprehensive overview of implementing the concepts discussed earlier to create a page object for use in a test scenario. By following this guide, you will gain a deeper insight into our approach to writing tests, enabling you to produce test code that is both maintainable and reusable.

#### Step 1: Define Selector Objects for NetworksView

Start by creating selector objects. These objects store identifiers for UI elements, making your tests easier to read and maintain. We'll use two types of selectors: IDs and Text. Let's call this file: `NetworksView.selectors.js`

```javascript
import enContent from '../../../locales/languages/en.json';

export const NetworksViewSelectorsIDs = {
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

#### Step 2: Create the Networks Page Object

The Networks Page Object encapsulates interactions with the NetworksView. It uses the selectors defined in _Step 1_ and utility functions for actions like typing and tapping. Methods within the page object hide the complexity of direct UI interactions, offering a simpler interface for test scripts. Use Matchers for finding elements and Gestures for performing actions, enhancing code reusability.

```javascript
import {
  NetworksViewSelectorsIDs,
  NetworkViewSelectorsText,
} from '../../selectors/Settings/NetworksView.selectors';
import Matchers from '../../utils/Matchers';
import Gestures from '../../utils/Gestures';

class NetworkView {
  get networkContainer() {
    return Matchers.getElementByID(NetworksViewSelectorsIDs.NETWORK_CONTAINER);
  }

  get addNetworkButton() {
    return device.getPlatform() === 'ios'
      ? Matchers.getElementByID(NetworksViewSelectorsIDs.ADD_NETWORKS_BUTTON)
      : Matchers.getElementByLabel(
          NetworksViewSelectorsIDs.ADD_NETWORKS_BUTTON,
        );
  }

  get rpcAddButton() {
    return device.getPlatform() === 'android'
      ? Matchers.getElementByLabel(
          NetworksViewSelectorsIDs.ADD_CUSTOM_NETWORK_BUTTON,
        )
      : Matchers.getElementByID(
          NetworksViewSelectorsIDs.ADD_CUSTOM_NETWORK_BUTTON,
        );
  }

  get customNetworkTab() {
    return Matchers.getElementByText(
      NetworkViewSelectorsText.CUSTOM_NETWORK_TAB,
    );
  }

  get rpcWarningBanner() {
    return Matchers.getElementByID(NetworksViewSelectorsIDs.RPC_WARNING_BANNER);
  }

  get rpcURLInput() {
    return Matchers.getElementByID(NetworksViewSelectorsIDs.RPC_URL_INPUT);
  }

  get networkNameInput() {
    return Matchers.getElementByID(NetworksViewSelectorsIDs.NETWORK_NAME_INPUT);
  }

  get chainIDInput() {
    return Matchers.getElementByID(NetworksViewSelectorsIDs.CHAIN_INPUT);
  }

  get networkSymbolInput() {
    return Matchers.getElementByID(
      NetworksViewSelectorsIDs.NETWORKS_SYMBOL_INPUT,
    );
  }

  async typeInNetworkName(networkName) {
    await Gestures.typeTextAndHideKeyboard(this.networkNameInput, networkName);
  }

  async typeInRpcUrl(rPCUrl) {
    await Gestures.typeTextAndHideKeyboard(this.rpcURLInput, rPCUrl);
  }

  async clearRpcInputBox() {
    await Gestures.clearField(this.rpcURLInput);
  }

  async tapAddNetworkButton() {
    await Gestures.waitAndTap(this.addNetworkButton);
  }

  async switchToCustomNetworks() {
    await Gestures.waitAndTap(this.customNetworkTab);
  }

  async typeInChainId(chainID) {
    await Gestures.typeTextAndHideKeyboard(this.chainIDInput, chainID);
  }

  async typeInNetworkSymbol(networkSymbol) {
    await Gestures.typeTextAndHideKeyboard(
      this.networkSymbolInput,
      networkSymbol,
    );
  }

  async tapRpcNetworkAddButton() {
    await Gestures.waitAndTap(this.rpcAddButton);
  }
}

export default new NetworkView();
```

#### Step 3: Utilize Page Objects in Test Specifications

With the page object in place, writing test specifications becomes more straightforward. The test scripts interact with the application through the page object's interface, improving readability and maintainability.

#### Example Test Case: Adding a Network

```javascript
import NetworkView from '../../pages/Settings/NetworksView';
import Assertions from '../../utils/Assertions';
import Networks from '../../resources/networks.json';

it('add Gnosis network', async () => {
  // Tap on Add Network button
  await NetworkView.tapAddNetworkButton();
  await NetworkView.switchToCustomNetworks();
  await NetworkView.typeInNetworkName(Networks.Gnosis.providerConfig.nickname);
  await NetworkView.typeInRpcUrl('abc'); // Negative test. Input incorrect RPC URL
  await Assertions.checkIfVisible(NetworkView.rpcWarningBanner);
  await NetworkView.clearRpcInputBox();
  await NetworkView.typeInRpcUrl(Networks.Gnosis.providerConfig.rpcUrl);
  await NetworkView.typeInChainId(Networks.Gnosis.providerConfig.chainId);
  await NetworkView.typeInNetworkSymbol(Networks.Gnosis.providerConfig.ticker);
  await NetworkView.tapRpcNetworkAddButton();
});
```

## TypeScript Implementation Guidelines for E2E Tests

The MetaMask Mobile e2e infrastructure supports TypeScript with a global types system that provides enhanced developer experience, type safety, and improved code maintainability.

Global Types Architecture
🎯 Global Type Definitions
All Detox element types are globally available in e2e files without imports via e2e/types/detox.d.ts:
```javascript
// ✅ Available globally - no imports needed
type DetoxElement = Promise<Detox.IndexableNativeElement | Detox.NativeElement | Detox.IndexableSystemElement>;
type TappableElement = Promise<Detox.IndexableNativeElement | Detox.SystemElement>;
type TypableElement = Promise<Detox.IndexableNativeElement>;

// Individual element types for specific casting
type IndexableNativeElement = Detox.IndexableNativeElement;
type NativeElement = Detox.NativeElement;
type SystemElement = Detox.SystemElement;

// Configuration and matcher types
type DeviceLaunchAppConfig = Detox.DeviceLaunchAppConfig;
type DetoxMatcher = Detox.NativeMatcher;
```

Best Practices
✅ Page Object Implementation
```javascript
// ✅ Good: Use global types without imports
class RequestPaymentView {
  get backButton(): TappableElement {
    return Matchers.getElementByID(RequestPaymentViewSelectors.BACK_BUTTON_ID);
  }

  get amountInput(): TypableElement {
    return Matchers.getElementByID(RequestPaymentViewSelectors.REQUEST_AMOUNT_INPUT_ID);
  }

  get containerElement(): DetoxElement {
    return Matchers.getElementByID(RequestPaymentViewSelectors.CONTAINER_ID);
  }
}
```

✅ Selector Files with Const Assertions
```javascript
// ✅ Good: Use const assertions for better type inference
export const RequestPaymentViewSelectorsIDs = {
  BACK_BUTTON_ID: 'request-payment-back-button',
  REQUEST_AMOUNT_INPUT_ID: 'request-amount-input',
  CONTAINER_ID: 'request-payment-container',
} as const;
```

✅ Test Specifications
```javascript
// ✅ Good: Proper async function typing
describe('Request Token Flow', (): void => {
  beforeAll(async (): Promise<void> => {
    await TestHelpers.reverseServerPort();
  });

  it('should request DAI amount', async (): Promise<void> => {
    await RequestPaymentView.tapOnToken();
    await Assertions.checkIfVisible(RequestPaymentView.amountInput);
  });
});
```

Migration Guidelines

🔄 Converting JavaScript Files to TypeScript
-  File Extensions: Change .js to .ts
-  Remove Explicit Type Imports: Global types eliminate the need for Detox type imports
-  Add Return Types: Use DetoxElement, TappableElement, or TypableElement for getters
-  Const Assertions: Add as const to selector objects
Before (JavaScript):
```javascript
// ❌ Old approach
import type { IndexableNativeElement } from 'detox/detox';

class MyPageView {
  get myButton() {
    return Matchers.getElementByID('button-id');
  }
}

export const Selectors = {
  BUTTON_ID: 'button-id',
};
```
After (TypeScript):
```javascript
// ✅ New approach - clean and type-safe
class MyPageView {
  get myButton(): TappableElement {
    return Matchers.getElementByID('button-id');
  }
}

export const Selectors = {
  BUTTON_ID: 'button-id',
} as const;
```

Developer Experience Benefits

🚀 IntelliSense and Autocomplete
-  Type Suggestions: Global types appear automatically when typing
-  Method Autocomplete: Full Detox method suggestions on typed elements
-  Error Prevention: Compile-time type checking prevents runtime errors
-  Parameter Hints: Detailed method parameter information

🔧 Enhanced Productivity
```javascript
// ✅ Full autocomplete available
get myButton(): TappableElement {
  return Matchers.getElementByID('button-id');
}

// When using myButton, you get autocomplete for:
// - .tap()
// - .longPress()
// - .multiTap()
// - All other Detox element methods
```

Type Safety Patterns

✅ Explicit Type Casting When Needed
```javascript
// ✅ For specific element types
await Assertions.checkIfElementToHaveText(
  ActivitiesView.transactionStatus(FIRST_ROW) as Promise<IndexableNativeElement>,
  'Confirmed',
  30000,
);
```

✅ Proper Async/Await Usage
```javascript
// ✅ Elements are promises, use await for interactions
const button = RequestPaymentView.submitButton;
await Gestures.waitAndTap(button);
await Assertions.checkIfVisible(button);
```

Verification and Testing

🧪 Type Checking
```javascript
# Verify TypeScript compilation
npx tsc --noEmit --project tsconfig.json

# Run e2e tests with TypeScript
yarn test:e2e:ios:debug:run e2e/specs/wallet/request-token-flow.spec.ts
```

Contributing Guidelines

When contributing TypeScript e2e tests:
- Follow Global Types: Use the global type system instead of explicit imports
- Maintain Compatibility: Ensure TypeScript files work alongside existing JavaScript files
- Add Type Annotations: Provide return types for element getters and test functions
- Use Const Assertions: Apply as const to selector objects for better type inference
- Test Thoroughly: Verify both compilation and runtime behavior
