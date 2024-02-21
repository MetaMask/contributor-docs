# MetaMask Mobile E2E Test Guidelines
 
 Writing Code Like a Well-Written Book 

A well-crafted piece of code is akin to a captivating story; it starts with a brilliant idea and elegantly unfolds to explain that idea in a coherent manner. Just as a well-written book is easy to read, your code should be a pleasure for anyone who peruses it. To ensure the readability of our code, let's delve into some key principles:

## 🗺 Locator Strategy

Good: The default strategy for locating elements is using Test IDs. However, in complex scenarios, employ Text or Label-based locators.

Bad: Overusing non-Test ID locators without justification.
 

### Locating Elements byID

✅ Good: To add Test IDs to a component, consult Detox's guidelines. Store Test IDs in separate '{PAGE}.selectors.js' files, corresponding to each page object. For example, 'PrivacyAndSecurity' should have 'PrivacyAndSecurity.selectors.js.'

### Locating Elements byText

✅ Good: When locating elements by text, retrieve the corresponding text string from the 'en.json' file in the 'locales/languages' folder. For instance, if you need to interact with "Show Hex Data," access it as follows:

```javascript

import en from '../../locales/languages/en.json';
en.app_settings.show_hex_data;
```

❌ Bad: Hardcoding text strings without retrieving them from the language file, like this:

```javascript

const elementText = "Show Hex Data"; // Hardcoded text
```
### Locating Elements byLabel

✅ Good: Utilize the 'byLabel()' element locator on Android when Test IDs aren't detected correctly.

❌ Bad: Unnecessarily relying on non-Test ID locators on all platforms, like this:


```javascript

if (device.getPlatform() === 'android') {
  await TestHelpers.waitAndTap('ANDROID-ELEMENT');
} else {
  await TestHelpers.waitAndTap('iOS-ELEMENT');
}
```

## 📌 Naming Convention

Good: Descriptive and clear names are the cornerstone of maintainable code.

Bad: Vague, cryptic names that leave readers puzzled.
Variables/Methods
✅ Good: Name variables and methods with utmost clarity. For an element like the "create wallet" button, the variable should be named "createWalletButton," and the method to interact with it should be named "tapCreateWalletButton."

❌ Bad: Using abbreviated or unclear names for variables and methods, like this:

```javascript
tapNTB() {
  await TestHelpers.waitAndTap(NTB_ID);
}
```
### Test IDs
When crafting Selector objects, it is crucial to name the object in a way that identifies the specific page where the elements are. The Selector object can be of type ID or string, and it is essential to explicitly specify this distinction. For instance, when creating selectors for the AddCustomToken view, planning to utilize both testIDs and strings as locator strategies, two separate Selector objects should be created.
 
When naming selector objects follow this pattern: 
{ScreenName}Selectors{Type}. In this case {TYPE} means selector type, i.e. IDs or Strings. 

✅ Good: 
```javascript

export const AddCustomTokenViewSelectorsIDs = {
};
export const AddCustomTokenViewSelectorsText = {
};
```

❌ Bad: Avoid generic names.
```javascript

export const customTokens = {
};
```

For selector keys, use uppercase letters to define them. Follow the format ELEMENT_NAME.

For instance:

✅ Good: 
```javascript

export const AddCustomTokenViewSelectorsIDs = {
  TOKEN_SYMBOL: ''
};
```
❌ Bad: 
```javascript

export const AddCustomTokenViewSelectorsIDs = {
  tokenSymbol: ''
};
```
For selector values, craft specific, lowercase strings using the format {screen}-{elementname}. 

For example:

✅ Good: 
```javascript

export const AddCustomTokenViewSelectorsIDs = {
  TOKEN_SYMBOL: 'token-screen-symbol'
};
```

❌ Bad: Avoid employing generic, ambiguous strings
```javascript

export const AddCustomTokenViewSelectorsIDs = {
  TOKEN_SYMBOL: 'symbol'
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
  IMPORT_BUTTON: 'add-token'
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

const DELETE_WALLET_INPUT_BOX_ID = 'delete-wallet-input-box'
```
## Assertions
✅ Good: Make assertion methods unmistakable by prefixing them with "is." This conveys whether an element or screen is visible. For example, 

```javascript

isCreateWalletButtonVisible()
```

❌ Bad: Using unclear or non-standard assertion method names, like this:
```javascript


checkButtonVisibility() {
  // Unclear method name
  // ...
}
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