# State Migration Best Practices

These best practices should apply to any team that contributes to an application that leverages local state.

## Best Practices
- When to Add a Migration: You should add a migration whenever you make a change to the shape of your state that is not backwards-compatible. This includes adding or removing properties, changing the type of a property, or moving properties around within the state tree (e.g. breaking controllers changes, etc...).
- Detecting Errors in a Migration: The best way to detect errors in a migration is through thorough testing. Write tests that take various shapes of old state and ensure that they are correctly transformed into the new state shape. Also, use TypeScript or another type system to help catch type errors.
- Handling Migration Errors: If a migration fails, you should have a strategy in place to handle the error. This could be as simple as logging the error and continuing with the default state, or it could involve more complex error recovery logic. In any case, it's important to ensure that your app can still function in some way even if a migration fails.
- Keeping Migrations Idempotent: Make sure your migrations are idempotent, meaning they can be run multiple times without changing the result beyond the initial application. This makes migrations safer and easier to test.
- Removing Old Migrations: Over time, you may accumulate many migrations. At some point, it may be safe to remove old migrations, especially if you know that all users have migrated to a newer version of the state. Be cautious with this, as removing a migration could break the ability to upgrade for any users who are still on old versions of the state.
