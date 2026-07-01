# Unit Testing Guidelines

## Use Jest

[Jest](https://jestjs.io/) is the testing framework for unit tests at MetaMask. Jest includes many built-in features that make it easier to write tests. Useful features include:

- [Module mocks](https://jestjs.io/docs/next/mock-functions#mocking-modules)
- [Timer mocks](https://jestjs.io/docs/next/timer-mocks)
- [Snapshots](https://jestjs.io/docs/next/snapshot-testing)
- Automatic [parallelization of tests](https://jestjs.io/blog/2016/09/01/jest-15#buffered-console-messages) without [restricting usage of exclusive tests](https://mochajs.org/#exclusive-tests-are-disallowed)
- A [consistent](https://jestjs.io/blog/2016/03/11/javascript-unit-testing-performance#adding-everything-up) [emphasis](https://jestjs.io/blog/2016/09/01/jest-15#new-cli-error-messages-and-summaries) on [great developer experience](https://jestjs.io/blog/2017/01/30/a-great-developer-experience)

## Colocate test files with implementation files

Place test files next to the code they test. This makes test files easier to find.

🚫

```
src/
  permission-controller.ts
test/
  permission-controller.ts
```

✅

```
src/
  permission-controller.ts
  permission-controller.test.ts
```

## Use `describe` to group tests for the same function/method

Wrap tests for the same function or method in a `describe` block. This provides three benefits:

- Tests are easier to find in large test files
- You can run only these tests using `.only`
- The test subject (the "it") is clear and focused

🚫

```typescript
describe('KeyringController', () => {
  it('adds a new account to the given keyring', () => {
    // ...
  });

  it('removes the account identified by the given address from its associated keyring', () => {
    // ...
  });
});
```

✅

```typescript
describe('KeyringController', () => {
  describe('addAccount', () => {
    it('adds a new account to the given keyring', () => {
      // ...
    });
  });

  describe('removeAccount', () => {
    it('removes the account identified by the given address from its associated keyring', () => {
      // ...
    });
  });
});
```

## 💡 Use `describe` to group tests under scenarios

When multiple tests verify the same conditional behavior, group them in a `describe` block. This way, you only need to specify the condition once. Start each test's name with `if ...` or `when ...` so it forms a complete sentence when combined with the `describe` block.

1️⃣

```typescript
it('delegates to the tokens controller when adding a token', () => {
  // ...
});

it('adds the new token to the state when adding a token', () => {
  // ...
});

it('returns true when adding a token', () => {
  // ...
});
```

2️⃣

```typescript
describe('when adding a token', () => {
  it('delegates to the tokens controller', () => {
    // ...
  });

  it('adds the new token to the state', () => {
    // ...
  });

  it('returns true', () => {
    // ...
  });
});
```

## Use `it` to specify the desired behavior for the code under test

[Each test must focus on a single aspect of the behavior](#keep-tests-focused), and its description must also describe that behavior clearly:

1. Start the description with an active verb in present tense (e.g., "returns", "displays", "prevents").
2. State the expected outcome or behavior directly.
3. Add contextual conditions only when necessary for clarity (e.g., "when session expires").
4. Keep descriptions concise but complete.
5. Avoid filler words ("should", "correctly", "successfully", "gracefully", "properly").
6. Avoid vague descriptions ("test", "edge case", "works", "handles").
7. Avoid implementation details ("calls redirectTo", "throws InvalidPayloadError").
8. Don't use the name of the function as the test description.

### Examples

🚫 **Using "should" unnecessarily**

```typescript
it('should successfully add token when address is valid and decimals are set and symbol exists', () => {
  // ...
});
```

✅ **Describe the behavior directly**

```typescript
it('stores valid token in state', () => {
  // ...
});
```

🚫 **Listing implementation details and parameters**

```typescript
it('should fail and show error message when invalid address is provided', () => {
  // ...
});
```

✅ **Focus on what is being tested**

```typescript
it('displays invalid address error', () => {
  // ...
});
```

🚫 **Stating obvious successful outcomes**

```typescript
it('works correctly when processing the transaction', () => {
  // ...
});
```

✅ **Be specific about the behavior**

```typescript
it('processes transaction', () => {
  // ...
});
```

🚫 **Describing implementation instead of behavior**

```typescript
it('calls redirectTo("/login") when session expires', () => {
  // ...
});
```

✅ **Describe the expected outcome**

```typescript
it('redirects to login when session expires', () => {
  // ...
});
```

🚫 **Using vague error language**

```typescript
it('throws an error when balance is insufficient', () => {
  // ...
});
```

✅ **Be precise about the expected behavior**

```typescript
it('prevents sending with insufficient balance', () => {
  // ...
});
```

Or, when the specific error type is the key behavior:

```typescript
it('throws InvalidPayloadError on malformed request', () => {
  // ...
});
```

🚫 **Missing or unclear description**

```typescript
it('test', () => {
  // ...
});

it('edge case', () => {
  // ...
});
```

✅ **Clear, descriptive names**

```typescript
it('returns empty array when input is empty', () => {
  // ...
});

it('accepts transaction up to maximum amount limit', () => {
  // ...
});
```

### Read more

- ["Tests as Specification"](http://xunitpatterns.com/Goals%20of%20Test%20Automation.html#Tests%20as%20Specification) and ["Tests as Documentation"](http://xunitpatterns.com/Goals%20of%20Test%20Automation.html#Tests%20as%20Documentation) in xUnit Patterns

## Keep tests focused

Tests are easier to understand and maintain when they cover only one aspect of behavior.

If you use "and" in a test description, the test is probably too large. Split it into separate tests.

🚫

```typescript
it('starts the block tracker and returns the block number', () => {
  // ...
});
```

✅

```typescript
it('starts the block tracker', () => {
  // ...
});

it('returns the block number', () => {
  // ...
});
```

## Don't directly test private code

Private code is not intended for consumers of an interface, so don't test it directly. Private code includes:

- Functions or classes not exported from a module
- Methods that start with `#` ([ECMAScript private fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields))
- Methods that start with `_` (older convention before private fields)
- Methods with the `private` keyword in TypeScript
- Functions or methods tagged with `@private` in TSDoc

Instead, test the public methods that call the private code, writing tests as if the private code were part of the public method.

The following example defines two private methods:

```typescript
// block-tracker.ts
import { request } from '...';

export class BlockTracker extends EventEmitter {
  isRunning = false;

  currentBlock: Block | null = null;

  subscriptionId: number | null;

  async stop() {
    if (!this.isRunning) {
      return;
    }
    this.isRunning = false;
    this.#startClearCurrentBlockTimer();
    await this.#end();
  }

  #startClearCurrentBlockTimer() {
    setTimeout(() => {
      this.currentBlock = null;
    }, 1000);
  }

  #end() {
    await request('eth_unsubscribe', this.subscriptionId);
  }
}
```

Since consumers (and tests) can only see the public interface, the example above behaves as if the private methods were inlined:

```typescript
// block-tracker.ts
import { request } from '...';

export class BlockTracker extends EventEmitter {
  isRunning = false;

  currentBlock: Block | null = null;

  subscriptionId: number | null;

  async stop() {
    if (!this.isRunning) {
      return;
    }
    this.isRunning = false;
    setTimeout(() => {
      this.currentBlock = null;
    }, 1000);
    await request('eth_unsubscribe', this.subscriptionId);
  }
}
```

A test suite for the class might look like this. Testing the public `stop` method verifies all the behaviors that result from calling it, including the effects of the private methods:

```typescript
describe('BlockTracker', () => {
  describe('stop', () => {
    it('does not reset the current block if the block tracker is stopped', () => {
      // ...
    });

    it('does not request to unsubscribe if the block tracker is stopped', () => {
      // ...
    });

    it('resets the current block if the block tracker is running', () => {
      // ...
    });

    it('requests to unsubscribe if the block tracker is running', () => {
      // ...
    });
  });
});
```

## Highlight the "exercise" phase

A test has up to four ["phases"](http://xunitpatterns.com/Four%20Phase%20Test.html):

1. **Setup:** Configure the environment to run the code under test
2. **Exercise:** Execute the code under test
3. **Verify:** Confirm that the code behaves as expected
4. **Teardown:** Return the environment to a clean state

Use empty lines to separate these phases visually. This helps readers understand the test flow.

In this example, the empty lines make it hard to see the relationships between parts. Which part executes the code? Which part confirms the behavior? Which part sets up the test?

1️⃣

```typescript
describe('KeyringController', () => {
  describe('submitEncryptionKey', () => {
    it('unlocks the keyrings with valid information', async () => {
      const keyringController = await initializeKeyringController({
        password: 'password',
      });
      keyringController.cacheEncryptionKey = true;
      const returnValue = await keyringController.encryptor.decryptWithKey();
      const decryptWithKeyStub = sinon.stub(
        keyringController.encryptor,
        'decryptWithKey',
      );
      decryptWithKeyStub.resolves(Promise.resolve(returnValue));

      keyringController.store.updateState({ vault: MOCK_ENCRYPTION_DATA });

      await keyringController.setLocked();

      await keyringController.submitEncryptionKey(
        'mockEncryptionKey',
        'mockEncryptionSalt',
      );

      expect(keyringController.encryptor.decryptWithKey.calledOnce).toBe(true);
      expect(keyringController.keyrings).toHaveLength(1);
    });
  });
});
```

This version is clearer. All setup code is grouped together visually, making the phases easy to identify:

2️⃣

```typescript
describe('KeyringController', () => {
  describe('submitEncryptionKey', () => {
    it('unlocks the keyrings with valid information', async () => {
      const keyringController = await initializeKeyringController({
        password: 'password',
      });
      keyringController.cacheEncryptionKey = true;
      const returnValue = await keyringController.encryptor.decryptWithKey();
      const decryptWithKeyStub = sinon.stub(
        keyringController.encryptor,
        'decryptWithKey',
      );
      decryptWithKeyStub.resolves(Promise.resolve(returnValue));
      keyringController.store.updateState({ vault: MOCK_ENCRYPTION_DATA });
      await keyringController.setLocked();

      await keyringController.submitEncryptionKey(
        'mockEncryptionKey',
        'mockEncryptionSalt',
      );

      expect(keyringController.encryptor.decryptWithKey.calledOnce).toBe(true);
      expect(keyringController.keyrings).toHaveLength(1);
    });
  });
});
```

### Read more

- ["Arrange, act, assert"](http://xp123.com/articles/3a-arrange-act-assert/) is a simplification of the four-phase test.
- In [behavior-driven development](https://www.agilealliance.org/glossary/bdd/), this is also called ["given, when, then"](https://martinfowler.com/bliki/GivenWhenThen.html).

## Keep tests isolated

A test must always pass. Running a test by itself or with a group of other tests must not change the result.

Tests must run in a clean environment. If a test changes any part of the environment, it must undo those changes before finishing. This prevents the test from affecting other tests.

The next sections suggest ways to keep tests isolated.

### Restore function mocks after each test

Set Jest's [`resetMocks`](https://jestjs.io/docs/configuration#resetmocks-boolean) and [`restoreMocks`](https://jestjs.io/docs/configuration#restoremocks-boolean) configuration options to `true`. Jest will then reset all mock functions and return them to their original implementations after each test. (MetaMask's [module template](https://github.com/MetaMask/metamask-module-template) includes this setting.)

<details><summary><b>Read more</b></summary>
<br/>

Mock functions that are visible to multiple tests must be reset properly. Otherwise, their state will affect other tests.

This example has two tests. The second test assumes the spy on `getNetworkStatus` from the first test is removed, but it is not:

🚫

```typescript
const optionsMock = {
  getNetworkStatus: () => 'available',
};

describe('token-utils', () => {
  it('returns null if the network is still loading', () => {
    // Oops! This changes the return value of this mock for every other test
    jest.spyOn(optionsMock, 'getNetworkStatus').mockReturnValue('loading');
    expect(getTokenDetails('0xABC123', optionsMock)).toBeNull();
  });

  it('returns the details about the given token', () => {
    // This will likely not work as `getNetworkStatus` still returns "loading"
    expect(getTokenDetails('0xABC123', optionsMock)).toStrictEqual({
      standard: 'ERC20',
      symbol: 'TEST',
    });
  });
});
```

You can save the spy to a variable and call `mockRestore` on it before the test ends:

```typescript
const optionsMock = {
  getNetworkStatus: () => 'available',
};

describe('token-utils', () => {
  it('returns null if the network is still loading', () => {
    const getNetworkStatusSpy = jest
      .spyOn(optionsMock, 'getNetworkStatus')
      .mockReturnValue('loading');
    expect(getTokenDetails('0xABC123', optionsMock)).toBeNull();
    getNetworkStatusSpy.mockRestore();
  });

  it('returns the details about the given token', () => {
    expect(getTokenDetails('0xABC123', optionsMock)).toStrictEqual({
      standard: 'ERC20',
      symbol: 'TEST',
    });
  });
});
```

However, the better approach is to set `resetMocks` and `restoreMocks` in your Jest configuration. Jest will then handle this automatically.

</details>

### Reset global variables

Create a helper function that wraps the code under test. This ensures changes to global variables are undone after use.

When possible, use [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) to pass globals to the code under test. This lets you pass mock implementations in tests.

<details><summary><b>Read more</b></summary>
<br/>

Global variables are properties of the global context (usually `global`). Changes to these variables affect every test in a test file.

If you want to change a global function in a test, mock it using `jest.spyOn`. This makes the mock easy to reset later.

🚫

```typescript
describe('NftDetails', () => {
  it('opens a tab', () => {
    // This change will apply to every other test that's run after this one
    global.platform = { openTab: jest.fn() };

    const { queryByTestId } = render(<NftDetails />);
    fireEvent.click(queryByTestId('nft-options__button'));

    await waitFor(() => {
      expect(global.platform.openTab).toHaveBeenCalledWith({
        url: 'https://testnets.opensea.io/assets/goerli/0xABC123/1',
      });
    });
  });

  it('renders the View Opensea button after opening a tab', () => {
    const { queryByTestId } = render(<NftDetails />);

    fireEvent.click(queryByTestId('nft-options__button'));

    // Oops! `global.platform.openTab` will still be a mock function, so
    // if this element is supposed to appear after opening a tab, it won't
    expect(queryByTestId('nft-options__view-on-opensea')).toBeInTheDocument();
  });
});
```

✅

```typescript
describe('NftDetails', () => {
  it('opens a tab', () => {
    // Now, as long as we set set Jest's `restoreMocks` configuration option to
    // true, this should work as expected
    jest.spyOn(global.platform, 'openTab').mockReturnValue();

    const { queryByTestId } = render(<NftDetails />);
    fireEvent.click(queryByTestId('nft-options__button'));

    await waitFor(() => {
      expect(global.platform.openTab).toHaveBeenCalledWith({
        url: 'https://testnets.opensea.io/assets/goerli/0xABC123/1',
      });
    });
  });

  it('renders the View Opensea button after opening a tab', () => {
    const { queryByTestId } = render(<NftDetails />);

    fireEvent.click(queryByTestId('nft-options__button'));

    // `global.platform.openTab` should no longer be mocked, so this will appear
    expect(queryByTestId('nft-options__view-on-opensea')).toBeInTheDocument();
  });
});
```

If you want to change a global property that is not a function, do not assign it directly in a test:

🚫

```typescript
describe('interpretMethodData', () => {
  it('returns the signature for setApprovalForAll on Sepolia', () => {
    // This change will apply to every other test that's run after this one
    global.ethereumProvider = new HttpProvider(
      'https://sepolia.infura.io/v3/abc123',
    );
    expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All',
    });
  });

  it('returns the signature for setApprovalForAll on Mainnet', () => {
    // Oops! This will still hit Sepolia
    expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All',
    });
  });
});
```

[Instead of using hooks](#avoid-before-each-and-after-each), create a function that wraps your test. This function captures the current global value before the test and restores it afterward:

1️⃣

```typescript
async function withEthereumProvider(
  ethereumProvider: Provider,
  test: () => void | Promise<void>,
) {
  const originalEthereumProvider = global.ethereumProvider;
  global.ethereumProvider = ethereumProvider;
  await test();
  global.ethereumProvider = originalEthereumProvider;
}

describe('interpretMethodData', () => {
  it('returns the signature for setApprovalForAll on Sepolia', () => {
    const sepolia = new HttpProvider('https://sepolia.infura.io/v3/abc123');
    withEthereumProvider(sepolia, () => {
      expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
        name: 'Set Approval For All',
      });
    });
  });

  it('returns the signature for setApprovalForAll on Mainnet', () => {
    // Now this test will "just work"
    expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All',
    });
  });
});
```

A better approach is to use [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) to remove the need for a global variable. This lets you create a fake value within your test:

2️⃣

```typescript
describe('interpretMethodData', () => {
  it('returns the signature for setApprovalForAll on Sepolia', () => {
    const provider = createFakeProvider({ network: 'sepolia' });
    expect(interpretMethodData(provider, '0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All',
    });
  });

  it('returns the signature for setApprovalForAll on Mainnet', () => {
    const provider = createFakeProvider({ network: 'mainnet' });
    expect(interpretMethodData(provider, '0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All',
    });
  });
});
```

</details>

### Reset shared variables

Use helper functions instead of variables to define data shared between tests.

<details><summary><b>Read more</b></summary>
<br/>

Variables declared outside of tests are not reset automatically between tests. Changes to these variables in one test can affect other tests. This breaks test isolation.

Example:

🚫

```typescript
const NETWORK = {
  provider: new HttpProvider('https://mainnet.infura.io/v3/abc123');
};

describe("interpretMethodData", () => {
  it("returns the signature for setApprovalForAll on Sepolia", () => {
    // This change will apply to every other test that's run after this one
    NETWORK.provider = new HttpProvider(
      'https://sepolia.infura.io/v3/abc123',
    );
    expect(interpretMethodData('0x3fbac0ab', NETWORK)).toStrictEqual({
      name: 'Set Approval For All'
    });
  });

  it("returns the signature for setApprovalForAll on Mainnet", () => {
    // Oops! This will still hit Sepolia
    expect(interpretMethodData('0x3fbac0ab', NETWORK)).toStrictEqual({
      name: 'Set Approval For All'
    });
  });
});
```

Using `beforeEach` might seem like a solution:

🚫

```typescript
describe("interpretMethodData", () => {
  const network;

  beforeEach(() => {
    // Now this variable is reset before each test
    network = {
      provider: new HttpProvider('https://mainnet.infura.io/v3/abc123');
    };
  });

  it("returns the signature for setApprovalForAll on Sepolia", () => {
    // This change will no longer apply to every other test
    network.provider = new HttpProvider(
      'https://sepolia.infura.io/v3/abc123',
    );
    expect(interpretMethodData('0x3fbac0ab', network)).toStrictEqual({
      name: 'Set Approval For All'
    });
  });

  it("returns the signature for setApprovalForAll on Mainnet", () => {
    // This will use Mainnet, as expected
    expect(interpretMethodData('0x3fbac0ab', network)).toStrictEqual({
      name: 'Set Approval For All'
    });
  });
});
```

[Instead of using hooks](#avoid-before-each-and-after-each), use a factory function. This function should define default values and let you override them when needed:

✅

```typescript
function buildNetwork({
  provider = HttpProvider('https://mainnet.infura.io/v3/abc123'),
}: Partial<{
  provider: HttpProvider;
}> = {}) {
  return { provider };
}

describe('interpretMethodData', () => {
  it('returns the signature for setApprovalForAll on Sepolia', () => {
    // Now `network` only lives for the duration of the test, so it cannot be
    // shared among other tests
    const network = buildNetwork({
      provider: new HttpProvider('https://sepolia.infura.io/v3/abc123'),
    });

    expect(interpretMethodData('0x3fbac0ab', network)).toStrictEqual({
      name: 'Set Approval For All',
    });
  });

  it('returns the signature for setApprovalForAll on Mainnet', () => {
    // This test will use Mainnet by default thanks to how we defined the helper
    const network = buildNetwork();

    expect(interpretMethodData('0x3fbac0ab', network)).toStrictEqual({
      name: 'Set Approval For All',
    });
  });
});
```

</details>

### Learn more

- [Discussion on C2 Wiki about isolating unit tests](https://wiki.c2.com/?UnitTestIsolation)

## Avoid the use of `beforeEach`

Extract shared setup steps into functions instead of putting them in a `beforeEach` hook:

🚫

```typescript
describe('TokenDetectionController', () => {
  let getCurrentChainId: jest.mock<Promise<string>, []>;
  let preferencesController: PreferencesController;
  let tokenDetectionController: TokenDetectionController;

  beforeEach(() => {
    getCurrentChainId = jest.fn().mockResolvedValue('0x1');
    preferencesController = new PreferencesController({
      getCurrentChainId,
    });
    tokenDetectionController = new TokenDetectionController({
      preferencesController,
    });
  });

  describe('constructor', () => {
    it('sets default state', () => {
      expect(tokenDetectionController.state).toStrictEqual({
        tokensByChainId: {},
      });
    });
  });

  describe('detectTokens', () => {
    it('tracks tokens for the currently selected chain', async () => {
      const sampleToken = { symbol: 'TOKEN', address: '0x2' };
      getCurrentChainId.mockResolvedValue('0x2');
      jest
        .spyOn(tokenDetectionController, 'fetchTokens')
        .mockResolvedValue(['0xAAA', '0xBBB']);

      await tokenDetectionController.detectTokens();

      expect(
        tokenDetectionController.state.tokensByChainId['0x2'],
      ).toStrictEqual(['0xAAA', '0xBBB']);
    });
  });
});
```

✅

```typescript
describe('TokenDetectionController', () => {
  describe('constructor', () => {
    it('sets default state', () => {
      const { controller } = buildTokenDetectionController();

      expect(controller.state).toStrictEqual({
        tokensByChainId: {},
      });
    });
  });

  describe('detectTokens', () => {
    it('tracks tokens for the currently selected chain', async () => {
      const sampleToken = { symbol: 'TOKEN', address: '0x2' };
      const { controller } = buildTokenDetectionController({
        getCurrentChainId: () => '0x2',
      });
      jest
        .spyOn(controller, 'fetchTokens')
        .mockResolvedValue(['0xAAA', '0xBBB']);

      await controller.detectTokens();

      expect(controller.state.tokensByChainId['0x2']).toStrictEqual([
        '0xAAA',
        '0xBBB',
      ]);
    });
  });
});

function buildTokenDetectionController({
  getCurrentChainId = () => '0x1',
}: {
  getCurrentChainId?: () => string;
}) {
  const preferencesController = new PreferencesController({
    getCurrentChainId,
  });
  const tokenDetectionController = new TokenDetectionController({
    preferencesController,
  });
  return { controller: tokenDetectionController, preferencesController };
}
```

<details><summary><b>Read more</b></summary>
<br/>

Using a `beforeEach` hook to set up tests with similar needs might seem convenient. However, this approach increases maintenance costs for two reasons:

- **It makes tests harder to read.** Different tests may need different setup, but `beforeEach` assigns equal importance to all setup steps. Readers must read all the setup code to find what matters for each test.
- **It makes writing new tests difficult.** The `beforeEach` setup may not fit new test scenarios. You may need complex refactoring to remove steps that don't apply, or use workarounds that hurt consistency and readability.

Setup helper functions solve these problems in two ways:

- Tests can pass options to the function to specify the setup they need, showing readers what is important for each test.
- Setup code is easier to refactor when needed.

Helper functions work for teardown too, not just setup. Consider the following example, which uses `beforeEach` to create a controller and `afterEach` to destroy it:

🚫

```typescript
describe('TokensController', () => {
  let controller: TokensController;
  let onNetworkDidChangeListener: (
    networkState: NetworkState,
  ) => void | undefined;

  beforeEach(() => {
    onNetworkDidChange = jest.fn();
    controller = new TokensController({
      chainId: '0x1',
      onNetworkDidChange: (listener) => {
        onNetworkDidChangeListener = listener;
      },
    });
  });

  afterEach(() => {
    controller.destroy();
  });

  describe('addToken', () => {
    it('registers the given token under the default network, assuming it has not changed yet', () => {
      controller.addToken({ symbol: 'DAI' });

      expect(controller.state.tokens).toStrictEqual({
        '0x1': {
          symbol: 'DAI',
        },
      });
    });

    it('registers the given token under the current network, even after it changes', () => {
      onNetworkDidChangeListener!({ chainId: '0x42' });

      controller.addToken({ symbol: 'DAI' });

      expect(controller.state.tokens).toStrictEqual({
        '0x42': {
          symbol: 'DAI',
        },
      });
    });
  });
});
```

A more maintainable pattern is a function that wraps the test, automatically creating and destroying the controller before and after it runs:

✅

```typescript
type WithControllerOptions = Partial<TokensControllerOptions>;

type WithControllerCallback = (setup: {
  controller: TokensController;
  onNetworkDidChangeListener: (networkState: NetworkState) => void;
}) => void | Promise<void>;

type WithControllerArgs =
  | [WithControllerOptions, WithControllerCallback]
  | [WithControllerCallback];

async function withController(...args: WithControllerArgs) {
  const [
    {
      chainId = '0x1',
      onNetworkDidChange = (listener) => {
        onNetworkDidChangeListener = listener;
      },
    },
    fn,
  ] = args.length === 1 ? [{}, args[0]] : args;
  const onNetworkDidChangeListener: (networkState: NetworkState) => void;

  const controller = new TokensController({ chainId, onNetworkDidChange });
  assert(onNetworkDidChangeListener, 'onNetworkDidChangeListener was not set');

  try {
    await fn({ controller, onNetworkDidChangeListener });
  } finally {
    controller.destroy();
  }
}

describe('TokensController', () => {
  describe('addToken', () => {
    it('registers the given token under the default network, assuming it has not changed yet', async () => {
      await withController({ chainId: '0x1' }, ({ controller }) => {
        controller.addToken({ symbol: 'DAI' });

        expect(controller.state.tokens).toStrictEqual({
          '0x1': {
            symbol: 'DAI',
          },
        });
      });
    });

    it('registers the given token under the current network, even after it changes', () => {
      await withController(({ controller, onNetworkDidChangeListener }) => {
        onNetworkDidChangeListener({ chainId: '0x42' });

        controller.addToken({ symbol: 'DAI' });

        expect(controller.state.tokens).toStrictEqual({
          '0x42': {
            symbol: 'DAI',
          },
        });
      });
    });
  });
});
```

</details>

## Keep critical data in the test

Tests often use data that is essential for setup or verification.

Keep this data inside the test instead of spreading it across the file or project. This makes the test easier to understand.

🚫

```typescript
const sampleMainnetTokenList = [
  {
    address: '0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f',
    symbol: 'SNX',
    decimals: 18,
    occurrences: 11,
    name: 'Synthetix',
    iconUrl: 'https://static.metafi.codefi.network/api/v1/tokenIcons/1/0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f.png',
    aggregators: ['Aave', 'Bancor', 'CMC'],
  },
];
const sampleMainnetTokensChainsCache = sampleMainnetTokenList.reduce(
  (output, current) => {
    output[current.address] = current;
    return output;
  },
  {} as TokenListMap,
);
const sampleSingleChainState = {
  tokenList: {
    '0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f': {
      address: '0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f',
      symbol: 'SNX',
      decimals: 18,
      occurrences: 11,
      name: 'Synthetix',
      iconUrl: 'https://static.metafi.codefi.network/api/v1/tokenIcons/1/0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f.png',
      aggregators: ['Aave', 'Bancor', 'CMC'],
    },
  };
};
const sampleTwoChainState = {
  tokensChainsCache: {
    '0x1': {
      timestamp,
      data: sampleMainnetTokensChainsCache,
    }
  },
}

// ... many, many lines later ...

describe('TokensController', () => {
  describe('start', () => {
    it('loads the token list for the selected chain', async () => {
      // The setup phase involves `sampleMainnetTokenList`...
      nock(TOKEN_END_POINT_API)
        .get(`/tokens/${convertHexToDecimal(ChainId.mainnet)}`)
        .reply(200, sampleMainnetTokenList);
      const controller = new TokenListController({
        chainId: ChainId.mainnet,
      });

      await controller.start();

      // ...and the verification phase involves `sampleSingleChainState` and
      // `sampleTwoChainState`. But it's not clear what they have to do with
      // `sampleMainnetTokenList` without scrolling all the way up and reading
      // all the way through the variables.
      expect(controller.state.tokenList).toStrictEqual(
        sampleSingleChainState.tokenList,
      );
      expect(
        controller.state.tokensChainsCache[ChainId.mainnet].data,
      ).toStrictEqual(
        sampleTwoChainState.tokensChainsCache[ChainId.mainnet].data,
      );
    });
  });
});
```

✅

```typescript
describe('TokensController', () => {
  describe('start', () => {
    it('loads the token list for the selected chain', async () => {
      // Now all of the data that the test ends up using in the execution and
      // verification phases is clearly spelled out. This also gives us an
      // opportunity to simplify the test setup.
      const chainIdInHex = '0x1';
      const chainIdInDecimal = '1';
      const tokensByAddress = {
        '0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f': {
          address: '0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f',
          symbol: 'SNX',
          decimals: 18,
          occurrences: 11,
          name: 'Synthetix',
          iconUrl: 'https://static.metafi.codefi.network/api/v1/tokenIcons/1/0xc011a73ee8576fb46f5e1c5751ca3b9fe0af2a6f.png',
          aggregators: ['Aave', 'Bancor', 'CMC'],
        };
      };
      nock(TOKEN_END_POINT_API)
        .get(`/tokens/${chainIdInDecimal}`)
        .reply(200, Object.values(tokensByAddress));
      const controller = new TokenListController({
        chainId: chainIdInHex,
      });

      await controller.start();

      expect(controller.state.tokenList).toStrictEqual(tokensByAddress);
      expect(
        controller.state.tokensChainsCache[chainIdInHex].data,
      ).toStrictEqual(tokensByAddress);
    });
  });
});
```

## Use Jest's mock functions instead of Sinon

Jest includes most of Sinon's features with a simpler API:

- Use `jest.fn()` instead of `sinon.stub()`.
- Use `jest.spyOn(object, method)` instead of `sinon.spy(object, method)` or `sinon.stub(object, method)`. (Note: The spied method will still be called by default.)
- Use `jest.useFakeTimers()` instead of `sinon.useFakeTimers()`. (Note: Jest's "clock" object had fewer features than Sinon's before Jest v29.5.)

## General manual mocks

Jest's documentation states: "Manual mocks are defined by writing a module in a `__mocks__/` subdirectory immediately". Jest automatically picks up these mocks for all tests. Be very careful when writing manual mocks because they are shared across all tests (including UI integration tests).

## Snapshots

Jest snapshots do not test whether a value is valid. They only check for changes since the last snapshot was created.

Do not consider snapshot matching as a full test of a component. Snapshots only verify that the component rendered without errors. The snapshot might show an error screen instead of the actual component.

Name your snapshot test cases clearly:

🚫 Wrong naming

```ts
describe('MyComponent', () => {
  it('should renders correctly', () => {
    // ...
  });
});
```

✅ Correct naming

```ts
describe('MyComponent', () => {
  it('matches rendered snapshot', () => {
    // ...
  });
});
```

You can use variants of this naming to add context. For example:

```ts
describe('MyComponent', () => {
  it('matches rendered snapshot when not enabled', () => {
    // ...
  });
});
```
