# Performance Tracing

## Overview

Performance tracing within MetaMask is the act of tracking the durations of user flows and code execution.

This can be debugged locally via developer builds, but also occurs in user production builds.

The resulting data is uploaded to Sentry where it can be aggregated, analysed, and trigger alerts to notify us of reduced performance.

### Automated Tracing

Automated tracing is provided by the `@sentry/browser` and `@sentry/react-native` packages.

They include the following integrations based on platform:

- [BrowserTracing](https://docs.sentry.io/platforms/javascript/configuration/integrations/browsertracing/)
- [ReactNativeTracing](https://docs.sentry.io/platforms/react-native/tracing/instrumentation/automatic-instrumentation/)

With no additional code, these automatically record the durations of operations such as:

- Browser Page Loads
- HTTP Callouts
- React Routing

### Custom Tracing

Custom tracing is when we manually update the code to invoke specific utility functions to record durations as we see fit to align with conceptual user flows such as:

- Ethereum Transaction Processing
- Signature Request Processing

Or alternatively to track more technical code stages such as:

- UI Initialisation

### Utilities

The [trace utilities](https://github.com/MetaMask/metamask-extension/blob/develop/shared/lib/trace.ts) provide an abstraction of the respective Sentry package to create Sentry transactions with minimum syntax, but maximum simplicity to support features such as:

- Tags
- Nested Traces
- Custom Timestamps

#### Trace Names

The `TraceName` enum provides a central definition of all traces in use within the MetaMask client.

It also simplifies the creation of traces that are started and stopped from different locations in the code.

### Examples

#### Trace the duration of a function.

```ts
import { trace, TraceName } from '...';

function someFunction() {
  return 1 + 1;
}

function someOtherFunction() {
  ...

  const value = trace(
    { name: TraceName.SomeTrace },
    someFunction
  );

  ...
}
```

#### Trace the duration of an asynchronous function.

The `trace` function automatically handles promises and records the duration until the promise is resolved or rejected.

```ts
import { trace, TraceName } from '...';

async function someFunction() {
  return new Promise(resolve => {
    ...
    resolve();
  });
}

async function someOtherFunction() {
  ...

  const value = await trace(
    { name: TraceName.SomeTrace },
    someFunction
  );

  ...
}
```

#### Trace the duration of a flow spanning multiple files.

```ts
// File1.ts

import { trace, TraceName } from '...';

function someFunction() {
  ...

  trace({ name: TraceName.SomeTrace });

  ...
}

// File2.ts

import { endTrace, TraceName } from '...';

function someOtherFunction() {
  ...

  endTrace({ name: TraceName.SomeTrace });

  ...
}
```

#### Generate a nested trace.

Nested traces are reflected in Sentry and allow the breakdown of a larger duration into smaller named durations.

```ts
import { trace, TraceName } from '...';

function someNestedFunction1() {
  return 1 + 1;
}

function someNestedFunction2() {
  return 2 + 2;
}

function someParentFunction(traceContext: TraceContext) {
  ...

  const value1 = trace(
    {
      name: TraceName.SomeNestedTrace1,
      parentContext: traceContext
    },
    someNestedFunction1
  );

  const value2 = trace(
    {
      name: TraceName.SomeNestedTrace2,
      parentContext: traceContext
    },
    someNestedFunction2
  );

  ...
}

function someOtherFunction() {
  ...

  trace(
    { name: TraceName.ParentTrace },
    (traceContext) => someParentFunction(traceContext)
  );

  ...
}

```

#### Include tags in a trace.

```ts
import { trace, TraceName } from '...';

function someFunction() {
  return 1 + 1;
}

function someOtherFunction() {
  ...

  const value = trace(
    {
      name: TraceName.SomeTrace
      tags: {
        someBoolean: true,
        someString: 'test',
        // Number tags are converted to Sentry measurements
        // to support search operations such as >= and <=.
        someNumber: 123
      }
    },
    someFunction
  );

  ...
}
```

#### Use manual timestamps in a trace.

```ts
import { trace, endTrace, TraceName } from '...';

...

trace({
  name: TraceName.SomeTrace,
  startTime: Date.now() - 123
});

...

endTrace({
  name: TraceName.SomeTrace,
  timestamp: Date.now() + 456
});

...
```
