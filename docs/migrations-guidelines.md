# Migration and Testing Guidelines

## Overview

This document outlines best practices and guidelines for writing migration scripts and their corresponding test cases.

The focus is on ensuring data integrity, handling errors gracefully, and maintaining consistency across our applications versions.

We always look for improvement, if you see anything that could be improved, please open a PR against this guidelines.

You can also check an example of a migration on MetaMask mobile app [here](https://github.com/MetaMask/metamask-mobile/blob/1855bd674e33bb0ece06fb6d8f09a4e5df46a108/app/store/migrations/044.ts#L1)

## Migration Guidelines

1. **State Integrity Checks**:

- State's on migrations are type `unknown`, it's crucial to validate state integrity before proceeding, we only migrate when structure and types meets our expectations,
- Validate the state and its nested properties using functions like `isObject` and `hasProperty` from `@metamask/utils`,
- Prevent data corruption by halting the migration on any inconsistencies and logging errors.

2. **Error Handling**:

The following prevents manipulation of potentially corrupted data, enforcing corrective action before proceeding.

- Log errors with `captureException` from Sentry, which is crucial for diagnosing issues post-migration,
- Ensure that error messages are descriptive: include the migration number and a clear description of the issue,
- If an exception is detected, indicating potential data corruption, halt the migration process and return the intial state,

3. **Return State**:

- Always return the state at the end of the migration function, whether it was modified or not,
- Returning the state ensures that the migration process completes and the state is passed to the next migrations.

## Testing Guidelines

1. **Initial State Setup**:

- Create an initial state that reflects possible real-world scenarios, including edge cases,
- if needed, create multiple initial states and use them each in a test for this specific case,

2. **Invalid State Scenarios**:

- Test how the migration handles invalid states, including null values, incorrect types, and missing properties,
- Ensure that the migration logs the appropriate errors without modifying and corrupting the state.

3. **Error Assertions**:

- Verify that errors are logged correctly for invalid states or unexpected conditions,

4. **Ensure State Immutability**:

- Always use deep cloning on the old state before passing it to the migration function in tests. For example, use `cloneDeep` from `lodash`.
  - Deep cloning preserves the integrity of your test data across different test cases,
  - Ensures the original state object is not mutated during the migration process,
  - guarantees that each test case runs on an correct, clean copy of the state.
- Never mutate the state directly as this can:
  - lead to hard-to-track bugs and false positives or negatives test results,
  - start subsequent tests with the original state as intended.
