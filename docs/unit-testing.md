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

## Use `describe` to group tests for the module under test

Wrap all tests for a file in a `describe` block named the same as the file itself:

🚫

``` typescript
// file-utils.ts

export function isFile() {
  // ...
}

export function isDirectory() {
  // ...
}

// file-utils.test.ts

describe('isFile', () => {
  // ...
});

describe('isDirectory', () => {
  // ...
});
```

✅

``` typescript
// file-utils.ts

export function isFile() {
  // ...
}

export function isDirectory() {
  // ...
}

// file-utils.test.ts

describe('file-utils', () => {
  describe('isFile', () => {
    // ...
  });

  describe('isDirectory', () => {
    // ...
  });
});
```

## Use `describe` to group tests for methods

Using `describe` to wrap tests for a method makes it easier to spot these tests in a large file and makes it possible to run them on their own (using `.only`). It also helps to establish the subject of the test (the "it") and therefore keeps the test well named and focused.

🚫

``` typescript
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

``` typescript
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

## 💡 Use `describe` to group tests for scenarios

If you have many tests that verify the behavior of a piece of code under a particular condition, you might find it helpful to wrap those tests with a `describe` block. Use a phrase starting with `if ...` or `when ...` so as to form a sentence when combined with the test name.

1️⃣

``` typescript
it('calls addToken on the tokens controller if "addToken" is true', () => {
  // ...
})

it('adds the new token to the state if "addToken" is true', () => {
  // ...
})

it('returns a promise that resolves to true if "addToken" is true', () => {
  // ...
})
```

2️⃣

``` typescript
describe('if "addToken" is true', () => {
  it('calls addToken on the tokens controller', () => {
    // ...
  });

  it('adds the new token to the state', () => {
    // ...
  });

  it('returns a promise that resolves to true', () => {
    // ...
  });
});
```

## Use `it` to specify known behavior for a unit of code

Use `it` to describe the behavior of a piece of code from the consumer's perspective as it stands today, not as it ought to be.

Do not use "should" at the start of the test name. The official Jest documentation [omits this word from their examples](https://jestjs.io/docs/next/getting-started), and it creates noise when reviewing the list of tests that Jest outputs after a run.

Do not repeat the name of the function or method in the name of the test.

### Examples

🚫

``` typescript
it("should not stop the block tracker", () => {
  // ...
});
```

✅

``` typescript
it("does not stop the block tracker", () => {
  // ...
});
```

🚫

``` typescript
describe('TokensController', () => {
  it("addToken", () => {
    // ...
  });
});
```

🚫

``` typescript
describe('TokensController', () => {
  it('adds a token', () => {
    // ...
  });
})
```

✅

``` typescript
describe('TokensController', () => {
  describe('addToken', () => {
    it('adds the given token to the "tokens" array in state', () => {
      // ...
    });
  });
})
```

## Keep tests focused

Tests are easier to reason about and maintain when they cover only one aspect of the code.

If you are using "and" in the name of a test, this could indicate the test involves too much behavior and may need to be broken up.

🚫

``` typescript
it("the block tracker is started and returns a promise that resolves to a block number", () => {
  // ...
});
```

✅

``` typescript
it("starts the block tracker", () => {
  // ...
});

it("returns a promise that resolves to a block number", () => {
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

``` typescript
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

``` typescript
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

``` typescript
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

Be aware of the way that a test moves through these phases, particularly if some steps like "exercise" and "verify" are repeated, as this may indicate that your test is doing too much.

It can be helpful when composing and reading tests to surround the "exercise" phase with empty line breaks. Observe:

1️⃣

``` typescript
describe('KeyringController', () => {
  describe('cacheEncryptionKey', () => {
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
  })
});
```

2️⃣

``` typescript
describe('KeyringController', () => {
  describe('cacheEncryptionKey', () => {
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
  })
});
```

### Read more

- ["Arrange, act, assert"](http://xp123.com/articles/3a-arrange-act-assert/) is a simplification of the four-phase test.
- In [behavior-driven development](https://www.agilealliance.org/glossary/bdd/), this is also called ["given, when, then"](https://martinfowler.com/bliki/GivenWhenThen.html).

## Keep tests isolated

A test must pass whether it is run individually or in concert with other tests, regardless of the order it appears in the test suite.

To achieve this, tests must be performed in a clean room. If a test makes changes to any part of the environment defined outside of itself, it must undo those changes before completing in order to prevent contaminating other tests.

Here are ways to do this:

### Restore function mocks after each test

Set Jest's [`resetMocks`](https://jestjs.io/docs/configuration#resetmocks-boolean) and [`restoreMocks`](https://jestjs.io/docs/configuration#restoremocks-boolean) configuration options to `true`. This instructs Jest to reset the state of all mock functions and return them to their original implementations after each test. (This option is set in MetaMask's [module template](https://github.com/MetaMask/metamask-module-template).)

<details><summary><b>Read more</b></summary>
<br/>

Care must be taken to ensure that mock functions that are visible to multiple tests in a test file are properly reset, otherwise the state of those mock functions can bleed over into other tests.

In this example, there are two tests. The second assumes that the spy on `getNetworkStatus` in the first test is removed, but that doesn't happen:

🚫

``` typescript
const optionsMock = {
  getNetworkStatus: () => 'available'
};

describe('token-utils', () => {
  it("returns null if the network is still loading", () => {
    // Oops! This changes the return value of this mock for every other test
    jest.spyOn(optionsMock, 'getNetworkStatus').mockReturnValue('loading');
    expect(getTokenDetails('0xABC123', optionsMock)).toBeNull();
  });

  it("returns the details about the given token", () => {
    // This will likely not work as `getNetworkStatus` still returns "loading"
    expect(getTokenDetails('0xABC123', optionsMock)).toStrictEqual({
      standard: 'ERC20',
      symbol: 'TEST'
    });
  });
})
```

Minimally, you can save the spy to a variable and then call `mockRestore` on it before the test ends:

``` typescript
const optionsMock = {
  getNetworkStatus: () => 'available'
};

describe('token-utils', () => {
  it("returns null if the network is still loading", () => {
    const getNetworkStatusSpy = jest
      .spyOn(optionsMock, 'getNetworkStatus')
      .mockReturnValue('loading');
    expect(getTokenDetails('0xABC123', optionsMock)).toBeNull();
    getNetworkStatusSpy.mockRestore();
  });

  it("returns the details about the given token", () => {
    // This will likely not work as `getNetworkStatus` still returns "loading"
    expect(getTokenDetails('0xABC123', optionsMock)).toStrictEqual({
      standard: 'ERC20',
      symbol: 'TEST'
    });
  });
})
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

``` typescript
describe("NftDetails", () => {
  it("opens a tab", () => {
    // This change will apply to every other test that's run after this one
    global.platform = { openTab: jest.fn() }

    const { queryByTestId } = render(<NftDetails />);
    fireEvent.click(queryByTestId('nft-options__button'));

    await waitFor(() => {
      expect(global.platform.openTab).toHaveBeenCalledWith({
        url: "https://testnets.opensea.io/assets/goerli/0xABC123/1",
      });
    });
  });

  it("renders the View Opensea button after opening a tab", () => {
    const { queryByTestId } = render(<NftDetails />);

    fireEvent.click(queryByTestId('nft-options__button'));

    // Oops! `global.platform.openTab` will still be a mock function, so
    // if this element is supposed to appear after opening a tab, it won't
    expect(queryByTestId('nft-options__view-on-opensea')).toBeInTheDocument();
  });
});
```

✅

``` typescript
describe("NftDetails", () => {
  it("opens a tab", () => {
    // Now, as long as we set set Jest's `restoreMocks` configuration option to
    // true, this should work as expected
    jest.spyOn(global.platform, "openTab").mockReturnValue();

    const { queryByTestId } = render(<NftDetails />);
    fireEvent.click(queryByTestId('nft-options__button'));

    await waitFor(() => {
      expect(global.platform.openTab).toHaveBeenCalledWith({
        url: "https://testnets.opensea.io/assets/goerli/0xABC123/1",
      });
    });
  });

  it("renders the View Opensea button after opening a tab", () => {
    const { queryByTestId } = render(<NftDetails />);

    fireEvent.click(queryByTestId('nft-options__button'));

    // `global.platform.openTab` should no longer be mocked, so this will appear
    expect(queryByTestId('nft-options__view-on-opensea')).toBeInTheDocument();
  });
});
```

Now say the global you want to change is not a function:

🚫

``` typescript
describe("interpretMethodData", () => {
  it("returns the signature for setApprovalForAll on Sepolia", () => {
    // This change will apply to every other test that's run after this one
    global.ethereumProvider = new HttpProvider(
      'https://sepolia.infura.io/v3/abc123',
    );
    expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All'
    });
  });

  it("returns the signature for setApprovalForAll on Mainnet", () => {
    // Oops! This will still hit Sepolia
    expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All'
    });
  });
});
```

[Instead of using hooks](#avoid-before-each-and-after-each), one idea is to create a function wrapping your test which captures the current value of the global beforehand and restores the global to this value afterward:

1️⃣

``` typescript
async function withEthereumProvider(ethereumProvider: Provider, test: () => void | Promise<void>) {
  const originalEthereumProvider = global.ethereumProvider;
  global.ethereumProvider = ethereumProvider;
  await test();
  global.ethereumProvider = originalEthereumProvider;
}

describe("interpretMethodData", () => {
  it("returns the signature for setApprovalForAll on Sepolia", () => {
    const sepolia = new HttpProvider('https://sepolia.infura.io/v3/abc123');
    withEthereumProvider(sepolia, () => {
      expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
        name: 'Set Approval For All'
      });
    });
  });

  it("returns the signature for setApprovalForAll on Mainnet", () => {
    // Now this test will "just work"
    expect(interpretMethodData('0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All'
    });
  });
});
```

Even better, however, is to make use of [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) to eliminate the need for a global. This allows you to create a fake version of the value in question within your test:

2️⃣

``` typescript
describe("interpretMethodData", () => {
  it("returns the signature for setApprovalForAll on Sepolia", () => {
    const provider = createFakeProvider({ network: "sepolia" });
    expect(interpretMethodData(provider, '0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All'
    });
  });

  it("returns the signature for setApprovalForAll on Mainnet", () => {
    const provider = createFakeProvider({ network: "mainnet" });
    expect(interpretMethodData(provider, '0x3fbac0ab')).toStrictEqual({
      name: 'Set Approval For All'
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

``` typescript
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

``` typescript
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

``` typescript
function buildNetwork({
  provider = HttpProvider('https://mainnet.infura.io/v3/abc123')
}: Partial<{
  provider: HttpProvider
}> = {}) {
  return { provider };
}

describe("interpretMethodData", () => {
  it("returns the signature for setApprovalForAll on Sepolia", () => {
    // Now `network` only lives for the duration of the test, so it cannot be
    // shared among other tests
    const network = buildNetwork({
      provider: new HttpProvider(
        'https://sepolia.infura.io/v3/abc123',
      ),
    });

    expect(interpretMethodData('0x3fbac0ab', network)).toStrictEqual({
      name: 'Set Approval For All'
    });
  });

  it("returns the signature for setApprovalForAll on Mainnet", () => {
    // This test will use Mainnet by default thanks to how we defined the helper
    const network = buildNetwork();

    expect(interpretMethodData('0x3fbac0ab', network)).toStrictEqual({
      name: 'Set Approval For All'
    });
  });
});
```
</details>

### Learn more

- [Discussion on C2 Wiki about isolating unit tests](https://wiki.c2.com/?UnitTestIsolation)

## Avoid `beforeEach` and `afterEach`

When writing tests that need to be set up or torn down in a similar way, it may be tempting to extract the setup and teardown phases to `beforeEach` and `afterEach` hooks:

🚫

``` typescript
describe('TokensController', () => {
  let controller: TokensController;
  let onNetworkDidChangeListener: (networkState: NetworkState) => void | undefined;

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

However, this strategy ends up increasing the long-term maintenance cost of tests for a couple of reasons:

- It makes debugging tests difficult later. Data that is critical to the purpose of a test — i.e., that is used in the execution or verification phase — may be hidden in a `beforeEach` hook, or it may be split across `beforeEach` and the test in question. This means that in order to trace data as it moves through the test, one must spend time piecing together the full picture.
- It makes writing tests for new scenarios difficult. The setup steps in `beforeEach` may not apply to future tests that cover different scenarios. In these cases, the author is forced to take on a complex refactor to remove the `beforeEach` steps that don't apply, or pursue complex workarounds.

A setup function serves the same purpose as hooks without causing these problems. With this pattern, tests that need to customize setup data can do so easily by passing options to the function, and this draws a clear line from the verification phase to the setup phase:

✅

``` typescript
type WithControllerOptions = Partial<TokensControllerOptions>;

type WithControllerCallback = (setup: {
  controller: TokensController;
  onNetworkDidChangeListener: (networkState: NetworkState) => void;
}) => void | Promise<void>;

type WithControllerArgs =
  | [WithControllerOptions, WithControllerCallback]
  | [WithControllerCallback];

async function withController(...args: WithControllerArgs) {
  const [{
    chainId = '0x1',
    onNetworkDidChange = (listener) => {
      onNetworkDidChangeListener = listener;
    },
  }, fn] = args.length === 1 ? [{}, args[0]] : args;
  const onNetworkDidChangeListener: (networkState: NetworkState) => void;

  const controller = new TokensController({ chainId, onNetworkDidChange });
  assert(onNetworkDidChangeListener, "onNetworkDidChangeListener was not set")

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

The only case in which `beforeEach` or `afterEach` is acceptable is when using the Jest API, such as when setting fake timers:

``` typescript
describe("some-module", () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  // ... tests go here ...
});
```

## Keep critical data in the test

Each test involves data that makes that test unique: the test may use special data to construct a scenario, render specific values to its environment, or call out to some other part of the system.

Keeping this data in the test instead of spread out across the file or project makes that test easier to follow from beginning to end.

🚫

``` typescript
const PASSWORD = 'abc123';

// ... many lines later ...

describe('KeyringController', () => {
  describe('submitPassword', () => {
    it('sets the password on the controller', async () => {
      const keyringController = await initializeKeyringController({
        password: PASSWORD,
      });
      await keyringController.submitPassword(PASSWORD);

      expect(keyringController.password).toBe(PASSWORD);
    });
  });
})
```

✅

``` typescript
describe('KeyringController', () => {
  describe('submitPassword', () => {
    it('sets the password on the controller', async () => {
      const keyringController = await initializeKeyringController({
        password: 'abc123',
      });
      await keyringController.submitPassword('abc123');

      expect(keyringController.password).toBe('abc123');
    });
  });
})
```

## Use Jest's mock functions instead of Sinon

Jest incorporates most of the features of Sinon with a slimmer API:

* `jest.fn()` can be used in place of `sinon.stub()`.
* `jest.spyOn(object, method)` can be used in place of `sinon.spy(object, method)` or `sinon.stub(object, method)` (with the caveat that the method being spied upon will still be called by default).
* `jest.useFakeTimers()` can be used in place of `sinon.useFakeTimers()` (though note that Jest's "clock" object had fewer features than Sinon's prior to Jest v29.5).
