# Unit Testing Guidelines

## Use Jest

[Jest](https://jestjs.io/) is the testing framework of choice for unit tests at MetaMask. It offers a more comprehensive set of features out of the box over Mocha or Tape that make writing tests for production code very easy. Some of the useful features are:

- [Module mocks](https://jestjs.io/docs/next/mock-functions#mocking-modules)
- [Timer mocks](https://jestjs.io/docs/next/timer-mocks)
- [Snapshots](https://jestjs.io/docs/next/snapshot-testing)
- Automatic [parallelization of tests](https://jestjs.io/blog/2016/09/01/jest-15#buffered-console-messages) without [restricting usage of exclusive tests](https://mochajs.org/#exclusive-tests-are-disallowed)
- A [consistent](https://jestjs.io/blog/2016/03/11/javascript-unit-testing-performance#adding-everything-up) [emphasis](https://jestjs.io/blog/2016/09/01/jest-15#new-cli-error-messages-and-summaries) on [great developer experience](https://jestjs.io/blog/2017/01/30/a-great-developer-experience)

## Colocate test files with implementation files

Test files are much easier to find when they sit next to the code they test.

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

Wrapping tests for the same function or method in a `describe` makes it easier to spot them in a large test file and makes it possible to run them on their own (using `.only`). It also helps to establish the subject of each test (the "it") and therefore keeps it well described and focused.

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

If you have many tests that verify the behavior of a piece of code under a particular condition, you might find it helpful to wrap them with a `describe` block so you only need to specify that condition once. Use a phrase starting with `if ...` or `when ...` so as to form a sentence when combined with the test description.

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

As each test [has to focus on a single aspect of that behavior](#keep-tests-focused), its description must describe that behavior using these guidelines:

1. Start with an active verb in present tense
2. Focus on what is being tested, not how
3. Be explicit about the context when needed
4. Keep names concise but descriptive
5. Avoid "should", "when", and other unnecessary words
6. Do not state obvious successful outcomes ("works successfully", "correctly")
7. Do not list test parameters; describe what they represent instead

### Examples

🚫 Don't

```typescript
it('should successfully add token when address is valid and decimals are set and symbol exists', () => {
  // ...
});

it('should fail and show error message when invalid address is provided', () => {
  // ...
});

it('works correctly when processing the transaction', () => {
  // ...
});

it('should throw error when balance is insufficient and user tries to send tokens', () => {
  // ...
});
```

✅ Do

```typescript
it('stores valid token in state', () => {
  // ...
});

it('displays invalid address error', () => {
  // ...
});

it('processes transaction', () => {
  // ...
});

it('prevents sending with insufficient balance', () => {
  // ...
});
```

The test description should communicate the expected behavior clearly and directly. Avoid:

- Repeating the name of the function or method being tested
- Using "should" at the beginning of the test name
- Including implementation details in the name
- Stating obvious successful outcomes
- Listing test parameters instead of what they represent
- Using words like "fail", "error", or "throw" when the error is the expected behavior

### Read more

- ["Tests as Specification"](http://xunitpatterns.com/Goals%20of%20Test%20Automation.html#Tests%20as%20Specification) and ["Tests as Documentation"](http://xunitpatterns.com/Goals%20of%20Test%20Automation.html#Tests%20as%20Documentation) in xUnit Patterns

## Keep tests focused

Tests are easier to reason about and maintain when they cover only one aspect of the intended behavior.

If you are using "and" in a test description, this could indicate the test is too large and may need to be broken up.

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

Private code is a part of an interface that is not intended to be used by consumers of that interface. This could mean:

- a function or class not exported from a module
- a method whose name starts with `#` (the syntax for [ECMAScript private fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields))
- a method whose name starts with `_` (an informal approach popular prior to private fields)
- a method preceded by the `private` keyword in TypeScript
- a function or method tagged with the `@private` JSDoc tag

Test code marked as private as though it were directly part of the public interface where it is executed. For instance, although you might write the following:

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

this is what consumers see:

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

and as a result, this is how the method ought to be tested:

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

## 💡 Highlight the "exercise" phase

A test can be subdivided into up to four kinds of actions, also called ["phases"](http://xunitpatterns.com/Four%20Phase%20Test.html):

1. **Setup:** Configuring the environment required to execute the code under test
2. **Exercise:** Executing the code under test
3. **Verify:** Confirming that the code under test behaves in an expected manner
4. **Teardown:** Returning the environment to a clean slate

Be aware of the way that a test moves through these phases. It can assist readers in following the flow by making judicious use of line breaks to separate phases visually.

For instance, the placement of empty lines in this test makes it a bit difficult to discern the relationships between different parts of the test. Which part executes the code under test, which part confirms the desired behavior, and which part just sets it up?

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

Now the different phases of this test are unmistakable, because all of the setup code is grouped together visually:

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

A test must pass whether it is run individually or whether it is run alongside other tests (in any order).

To achieve this, tests must be performed in a clean room. If a test makes changes to any part of the environment defined outside of itself, it must undo those changes before completing in order to prevent contaminating other tests.

Here are ways to do this:

### Restore function mocks after each test

Set Jest's [`resetMocks`](https://jestjs.io/docs/configuration#resetmocks-boolean) and [`restoreMocks`](https://jestjs.io/docs/configuration#restoremocks-boolean) configuration options to `true`. This instructs Jest to reset the state of all mock functions and return them to their original implementations after each test. (This option is set in MetaMask's [module template](https://github.com/MetaMask/metamask-module-template).)

<details><summary><b>Read more</b></summary>
<br/>

Care must be taken to ensure that mock functions that are visible to multiple tests in a test file are properly reset, otherwise the state of those mock functions can bleed over into other tests.

In this example, there are two tests. The second assumes that the spy on `getNetworkStatus` in the first test is removed, but that doesn't happen:

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

Minimally, you can save the spy to a variable and then call `mockRestore` on it before the test ends:

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
    // This will likely not work as `getNetworkStatus` still returns "loading"
    expect(getTokenDetails('0xABC123', optionsMock)).toStrictEqual({
      standard: 'ERC20',
      symbol: 'TEST',
    });
  });
});
```

But it is better to let Jest take care of this for you by setting `resetMocks` and `restoreMocks` in your Jest configuration.

</details>

### Reset global variables

Create a helper function that wraps the code under test to ensure that changes to globals get undone after they are used.

Where it makes sense, use [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) to pass globals to the code under test so that a mock implementation can be passed in tests.

<details><summary><b>Read more</b></summary>
<br/>

Global variables are equivalent to properties of the global context (usually `global`). Changing these variables will naturally affect every test in a test file.

If the global you want to change is a function, it is best to mock the function using `jest.spyOn` so that it is easy to undo it later (note: this guideline is best paired with the above guideline so that you can make this happen automatically):

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

Now say the global you want to change is not a function:

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

[Instead of using hooks](#avoid-before-each-and-after-each), one idea is to create a function wrapping your test which captures the current value of the global beforehand and restores the global to this value afterward:

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

Even better, however, is to make use of [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) to eliminate the need for a global. This allows you to create a fake version of the value in question within your test:

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

Variables declared outside of tests are not reset automatically between tests. Thus, changes made to these variables in tests can affect later tests, breaking test isolation.

For example:

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

It is tempting to reach for `beforeEach` to correct this:

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

[Instead of using hooks](#avoid-before-each-and-after-each), however, use a factory function which defines defaults and allows them to be overridden where they are needed:

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

Extract setup steps shared among multiple tests to functions instead of lumping them into a `beforeEach` hook:

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

When writing tests that need to be set up in a similar way, it may be tempting to use a `beforeEach` hook. However, this strategy ends up increasing the long-term maintenance cost of tests for a couple of reasons:

- It makes tests harder to read. Tests that run under different scenarios may require different ways of being set up, but using a `beforeEach` hook unnecessarily assigns the same importance to all setup steps listed there. This forces the reader to peruse all of the setup code in order to distinguish the signal from the noise.
- It makes writing tests for new scenarios difficult. The setup steps specified in the `beforeEach` section may not perfectly apply to future tests that cover different scenarios. In these cases, the author may be forced to take on a complex refactor to remove the `beforeEach` steps that don't apply or pursue workarounds which decrease consistency and readability.

Setup helper functions serve the same purpose as hooks without causing these problems. In this way, tests that need setup data for specific scenarios can specify them easily by passing options to the function, which communicates their importance to readers over other kinds of setup data in the context of the test. Plus, managing setup code becomes easier, as it can be refactored more easily if necessary.

The functional pattern presented above can not only be applied to setup code but also teardown code. In that case, your helper becomes a wrapper by taking another function:

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

A test may involve data that is essential for the test to set up a scenario and/or verify the intended behavior.

Keeping this data inside of the test instead of spread out across the file or project makes the "story" that the test is telling easier to follow.

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

Jest incorporates most of the features of Sinon with a slimmer API:

- `jest.fn()` can be used in place of `sinon.stub()`.
- `jest.spyOn(object, method)` can be used in place of `sinon.spy(object, method)` or `sinon.stub(object, method)` (with the caveat that the method being spied upon will still be called by default).
- `jest.useFakeTimers()` can be used in place of `sinon.useFakeTimers()` (though note that Jest's "clock" object had fewer features than Sinon's prior to Jest v29.5).

## Avoid general manual mocks:

According to Jest's documentation `Manual mocks are defined by writing a module in a __mocks__/ subdirectory immediately`. These types of mocks are automatically picked up by Jest for all tests. We should be very careful when writing this types of mocks as they will be shared across all tests (including UI integration tests).

## Snapshots

Jest snapshots are not testing the validity of the value tested against a snapshot.
It only checks for changes since last snapshot generation.

Never consider that rendering a component and matching the snapshot is a test of the component:
It will only check that the component render worked.
It may be an error screen and not the actual component.

To make this clear, name your snapshot test cases by the following rules:

🚫 Wrong naming

```ts
describe('MyComponent', () => {
  it('should renders correctly')
```

✅ Correct naming

```ts
describe('MyComponent', () => {
  it('matches snapshot')
```

Of course variants of this naming can be used to add some context, for instance:

```ts
describe('MyComponent', () => {
   it('render matches snapshot when not enabled'
```
