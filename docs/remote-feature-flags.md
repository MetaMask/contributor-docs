# Remote Feature Flags

## Overview

This document outlines the process for adding and managing remote feature flags in MetaMask (Extension and Mobile). Remote feature flags allow us to control feature availability and behavior in production without requiring a new release. They are created on the LaunchDarkly platform and distributed by our internal API [ClientConfigAPI](https://github.com/consensys-vertical-apps/mmwp-client-config-api). Please read the [Remote feature flags ADR](https://github.com/MetaMask/decisions/pull/43) for more details.

### Choosing the Right Feature Flag Type

Choose the appropriate feature flag type based on your needs:

**1. Boolean flag**: Use when you need a simple ON/OFF toggle for a feature. In this example: `my-feature` is enabled for all users.

```json
{
  "name": "my-feature",
  "value": true
}
```

**2. Object flag with scope based on "threshold"**: Use for:

- controlling % of users who see a feature
  In this example: the feature is enabled for 30% of users and disabled for 70% of users.

```json
[
  {
    "name": "feature is ON",
    "scope": {
      "type": "threshold",
      "value": 0.3
    },
    "value": true
  },
  {
    "name": "feature is OFF",
    "scope": {
      "type": "threshold",
      "value": 1
    },
    "value": false
  }
]
```

- A/B testing different feature variants
  In this configuration, the swap button will be blue for 25% of users, red for 25% of users, green for 25% of users, and yellow for 25% of users

```json
[
  {
    "name": "blue",
    "scope": {
      "type": "threshold",
      "value": 0.25
    },
    "value": "0000FF"
  },
  {
    "name": "red",
    "scope": {
      "type": "threshold",
      "value": 0.5
    },
    "value": "FF0000"
  },
  {
    "name": "green",
    "scope": {
      "type": "threshold",
      "value": 0.75
    },
    "value": "00FF00"
  },
  {
    "name": "yellow",
    "scope": {
      "type": "threshold",
      "value": 1
    },
    "value": "FFF500"
  }
]
```

The distribution is deterministic based on the user's `metametricsId`, ensuring consistent group assignment across sessions.

## Implementation Guide

### 1. Creating feature flag in LaunchDarkly.

1. Name your flag with team prefix (e.g., `confirmations-blockaid`, `ramps-card`)
2. Request LaunchDarkly access from TechOps. Here's a template email:

```
Subject: Requesting access to LaunchDarkly for [team]

Dear TechOps,
Can I please get access to LaunchDarkly with the "Metamask Extension and Mobile" role? (role key: `metamask-config`)

Thanks!
```

3. Select appropriate project in LaunchDarkly:
   - Extension: "MetaMask Client Config API - Extension"
   - Mobile: "MetaMask Client Config API - Mobile"
4. Create the flag following the [LaunchDarkly flag creation guide](https://docs.launchdarkly.com/home/flags/new)

### 2. Using Remote Feature Flags in MetaMask clients

#### Extension

To use a remote feature flag in the MetaMask extension, follow these steps:

1. Add selector from `import { getRemoteFeatureFlags } from 'path/to/selectors';` in your component file.
2. Use the selector to get the feature flag value.

```typescript
const { testFeatureFlag } = getRemoteFeatureFlags(state);
```

3. Use the feature flag value in your component.

```typescript
if (testFeatureFlag) {
  // Use the feature flag value
}
```

#### Mobile

##### Create a selector for your feature flag

- Location: `app/selectors/featureFlagsController`
- Create a new folder with your feature flag name
- Follow the structure from `app/selectors/featureFlagsController/minimumAppVersion`

Your selector must include:

- State selectors for each feature flag value
- Fallback values for invalid remote flag values
- TypeScript types
- Unit tests
- Any required business logic for value manipulation

> [!IMPORTANT]
> Always use the specific feature flag selector when accessing values. Accessing feature flags directly from Redux state or the main selector bypasses fallback values and can cause crashes.

### 3. Testing Remote Feature Flags

#### Extension

##### Local Feature Flag Override

- Developers can override `remoteFeatureFlag` values by defining them in `.manifest-overrides.json` and enable `MANIFEST_OVERRIDES=.manifest-overrides.json` in the `.metamaskrc.dist` locally.
- These overrides take precedence over values received from the controller
- Accessible in all development environments:
  - Local builds
  - Webpack builds
  - E2E tests

##### Developer Validation

##### A. Local Build

1. Define overrides in `remoteFeatureFlags` in `manifest-overrides.json`:

```json
{
  "_flags": {
    "remoteFeatureFlags": {
      "testFlagForThreshold": {
        "name": "test-flag",
        "value": "121212"
      }
    }
  }
}
```

2. Verify in Developer Options:
   - Open extension
   - Click on the account menu (top-right corner)
   - Select "Settings" > "Developer Options"
   - Look for "Remote Feature Flags" section to verify your overrides

##### B. E2E Test

Add the customized value in your test configuration:

```typescript
await withFixtures({
  fixtures: new FixtureBuilder()
    .withMetaMetricsController({
      metaMetricsId: MOCK_META_METRICS_ID,
      participateInMetaMetrics: true,
    })
    .build(),
  manifestFlags: {
    remoteFeatureFlags: MOCK_CUSTOMIZED_REMOTE_FEATURE_FLAGS,
  },
  title: this.test?.fullTitle(),
});
```

#### Mobile

##### Local Feature Flag Override

1. Set environment variable in `.js.env` and run the app. This forces the fallback values to override any remote values.

```
OVERRIDE_REMOTE_FEATURE_FLAGS=TRUE
```

2. Test different feature flag scenarios:
   - Modify the default values in your feature flag selector
   - Run the application to verify behavior with different values
   - Remember to test both override and non-override scenarios
