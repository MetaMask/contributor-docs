# Migration and Testing Guidelines

## Overview

This document outlines best practices and guidelines for writing migration scripts and their corresponding test cases.

The focus is on ensuring data integrity, handling errors gracefully, and maintaining consistency across mobile app versions.

## Migration Guidelines

1. **Pre-Validation Checks**:

- Ensure the state to be migrated is valid before running the migration logic,
- Verify the version and integrity of the state with utility functions like `ensureValidState`.

2. **Error Handling**:

The following prevents manipulation of potentially corrupted data, enforcing corrective action before proceeding.

- Log erros with `captureException` from `@sentry/react-native`, which is crucial for diagnosing issues post-migration,
- Ensure that error messages are descriptive: include the migration number and a clear description of the issue,
- If an exception is detected, indicating potential data corruption, halt the migration process and return the intial state,

3. **State Integrity Checks**:

- Checks the state's structure and types carefully,
- Validate the state and its nested properties using functions like `isObject` and `hasProperty`,
- Prevent data corruption by halting the migration on any inconsistencies and logging errors.

5. **Avoid Type Casting**:

- Do not use type casting (for example, `as` keyword in TypeScript) to manipulate the state or its properties,
- Type casting can mask underlying data structure issues and make the code less resilient in case of change in dependencies or data structures,
- Instead, use type guards and runtime checks to ensure data integrity and compatibility.

6. **Return State**:

- Always return the state at the end of the migration function, whether it was modified or not,
- Returning the state ensures that the migration process completes and the state is passed to the next migrations.

## Testing Guidelines

1. **Mocking**:

- Mock dependencies with `jest.mock` (for example `@sentry/react-native`),
- Mocking isolates the migration under test and prevents side effects.

2. **Initial State Setup**:

- Create an initial state that reflects possible real-world scenarios, including edge cases,
- if needed, create multiple initial states and use them each in a test for this specific case,
- Use utilities like `merge` from `lodash` to combine base states with specific test case modifications.

4. **Invalid State Scenarios**:

- Test how the migration handles invalid states, including null values, incorrect types, and missing propertie,
- Ensure that the migration logs the appropriate errors without modifying and corrupting the state.

5. **Data Synchronization Tests**:

- Write tests that verify the correct synchronization of data for migrations that synchronize data between controllers,
- Check the presence and accuracy of the synchronized data.

6. **Error Assertions**:

- Verify that errors are logged correctly for invalid states or unexpected conditions,
- Use `expect` to assert that `captureException` was called with the expected error messages.

8. **Ensure State Immutability**:

- Always use deep cloning on the old state before passing it to the migration function in tests. For example, use `cloneDeep` from `lodash`.
  - Deep cloning preserves the integrity of your test data across different test cases,
  - Ensures the original state object is not mutated during the migration process,
  - guarantees that each test case runs on an correct, clean copy of the state.
- Never mutate the state directly as this can:
  - lead to hard-to-track bugs and false positives or negatives test results,
  - start subsequent tests with the original state as intended.

## Conclusion

You will improve migrations robustness by following these guidelines and enforce error-resistance, and maintain data integrity.

You will identify more potential issues with extended testing of migrations before they reach our users.

Always aim for clarity and thoroughness in both migration scripts and tests.
