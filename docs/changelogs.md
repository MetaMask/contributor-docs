# Guide to Changelogs

Changelogs are invaluable for capturing and communicating releases made to projects over time. Maintaining changelogs effectively is undeniably a bit of an art, but a lot of science goes into it as well, and this document aims to guide engineers accordingly.

## tl;dr

- Every MetaMask project for which new versions are distributed publicly should have a changelog file, and it should be called `CHANGELOG.md`.
- A changelog should be written primarily for consumers of the project, and as such, it should be valuable to them at all times.
- A changelog is valuable when it clearly and concisely describes modifications that have been made to the surface area of the project at each version throughout time, highlighting special versions that require consumers to make changes to _their_ project to avoid problems or migrate to a different workflow to be able to continue to use the software effectively.
- The surface area of a project include the parts of the API, CLI, and/or GUI that consumers can "see" and use, as well as the external code that it relies on.
  - The API of a project covers exported code and data, and may include classes, methods, functions, constants, and types.
  - The CLI of a project covers executables, and may include commands, options, and workflows.
  - The GUI of a project covers applications, and may include screens, interactive components, and workflows.
  - The external code of a project is its dependencies or peer dependencies.

## Keep and enforce a changelog

Projects that use a release process to deploy new versions of the project to a publicly accessible location such as NPM or GitHub Pages should include a file in the repo that captures those releases and the changes included in each release. This file should be called `CHANGELOG.md`.

The changelog should follow a [standard format](#understand-the-format-of-the-changelog). The [`@metamask/auto-changelog`](https://github.com/MetaMask/auto-changelog) tool should be installed in the repo to ensure that the format is followed at all times:

- A `lint:changelog` package script should be present which runs `auto-changelog validate`
- A `lint` package script should be present which runs `lint:changelog`
- CI should be configured to run `lint` on all branches and prevent a PR from being merged if `lint` does not pass

See the [module template](https://github.com/MetaMask/metamask-module-template) for an example of a project that does this.

## Understand the format and structure of the changelog

All changelogs should be written in a format based on the ["Keep a Changelog"](https://keepachangelog.com/) specification. This format is enforced by [`@metamask/auto-changelog`](https://github.com/MetaMask/auto-changelog).

Here is an example for a fictitious `@metamask/logger` package:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Add support for logging strategies ([#343](https://github.com/MetaMask/logger/pull/343))
  - The `logLevels` option to the `Logger` constructor may now be an object instead of an array, where each value is a function that takes a `message` and does something with it.
  - You could use this, for instance, to log `debug` messages to the console but log `error` messages to Sentry.

## [1.1.0]

### Added

- Add support for log level `info` ([#270](https://github.com/MetaMask/logger/pull/270))
  - The `Logger` constructor now supports `info` as a possible value to the `logLevels` option.
  - Add `"info"` to the `LogLevel` type union.

### Changed

- Upgrade `loglevel` from `^3.1.0` to `^3.4.2` ([#211](https://github.com/MetaMask/logger/pull/211))

## [1.0.0]

### Added

- Initial release

[Unreleased]: https://github.com/MetaMask/logger/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/MetaMask/logger/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/MetaMask/logger/releases/tag/v1.0.0
```

Broadly, the changelog has the following structure:

1. Prelude, the top-level header and introduction
2. "Unreleased", a special section representing unpublished changes
3. One or more sections representing published versions and their changes
4. Link references for section headers

The "Unreleased" section and version-specific sections begin with a header (either "Unreleased" or a version). Within each of these sections is set of categories. Although it is not necessary to provide every category, when present they must appear in the following order (enforced by `@auto-changelog` [here](https://github.com/MetaMask/auto-changelog/blob/ef3e86e15b0de7061856a53fd18c4f38e898f5e8/src/constants.ts)). Note that this diverges from the "Keep a Changelog" spec:

1. Uncategorized
2. Added
3. Changed
4. Deprecated
5. Removed
6. Fixed
7. Security

Finally, within each category section is a bulleted list of changelog entries. Each changelog entry must end with a link to the pull request that introduced the change. If necessary, another bulleted list may be placed under a changelog entry to explain its purpose and provide more details for usage or adoption.

## List unreleased changes separately from released changes

Entries for changes that have not been released should be filed under a section at the top of the file called "Unreleased". (When a new release is issued, the entries here will automatically be moved to a new section by `@metamask/auto-changelog`.)

## Place changes into categories

Before adding a new changelog entry, determine which category it belongs to, adding a new header for the category if it does not exist. Do not leave changes in "Uncategorized".

Consult the [format](#understand-the-format-of-the-changelog) for the available list of change categories and the order in which they should appear.

### Added

A change should be filed under this category if it enables consumers to do or use something with the project that they couldn't before.

Most additions are non-breaking, but see ["Highlight breaking changes"](#highlight-breaking-changes) for exceptions.

This could include:

- Adding a new package export (e.g., a new class, function, method, or type) ✅
- Adding a new method to an exported class ✅
- Adding a new argument to an exported function or a method of an exported class (as long as it doesn't change the signature in a non-compatible way) ✅
- Adding a new option to an executable ✅
- Adding a new subcommand to a executable ✅
- Adding a new executable entirely ✅
- Adding a new screen or workflow to a GUI ✅

Notably, this does not include:

- Adding a new "production" or peer dependency (since consumers can't use dependencies directly) ❌
  - This should be filed under "Changed"
- Adding a new development dependency ❌
  - This should be excluded from the changelog entirely, since it is only relevant for developers
- Adding a new value to a TypeScript type union or enum ❌
  - This should be filed under "Changed"
- Adding a new property to a TypeScript type ❌
  - This should be filed under "Changed"
- Adding a new value to an array, map, or other data structure which is exported directly ❌
  - This should be filed under "Changed"
- Adding a new class, function, method, etc. that is not being exported ❌
  - This should be excluded from the changelog entirely, since only developers can "see" it

### Removed

A change should be filed under this category if it removes an ability for consumers to do or use something with the project that they previously had.

In most cases, these kinds of changes should be marked as breaking (see ["Highlight breaking changes"](#higlight-breaking-changes) for the full list).

This could include:

- Removing a package export (e.g., a new class, function, method, or type) ✅
- Removing a new method from an exported class ✅
- Removing an argument from an exported function or a method of an exported class ✅
- Removing an option from an executable ✅
- Removing a subcommand from an executable ✅
- Removing an executable entirely ✅
- Removing a new screen or workflow from a GUI ✅

Notably, this does not include:

- Removing a "production" or peer dependency (since consumers can't use dependencies directly) ❌
  - This should be filed under "Changed"
- Removing a development dependency ❌
  - This should be excluded from the changelog, since it is only relevant for developers
- Removing a value from a TypeScript type union or enum ❌
  - This should be filed under "Changed"
- Removing a property from a TypeScript type ❌
  - This should be filed under "Changed"
- Removing a value from an array, map, or other data structure which is exported directly ❌
  - This should be filed under "Changed"
- Removing a class, function, method, etc. that is not being exported ❌
  - This should be excluded from the changelog entirely, since only developers can "see" it

### Fixed

A change should be filed under this category if it corrects a problem that was introduced in a previous version.

This could include:

- Correcting confusing or illogical behavior ✅
- Making a change that was advertised in a previous release but was accidentally left out ✅
- Changing the TypeScript signature of a function or method to match existing runtime validation ✅

Notably, this does not include:

- Upgrading a dependency to patch a security vulnerability
  - This should be filed under "Security"

### Deprecated

A change should be filed under this category if it adds a warning for a part of the API that is planned to be removed in a future version (but has not been removed yet).

### Security

A change should be filed under this category if it patches a publicly known security vulnerability (usually one that has been assigned a ID in the CVE database and highlighted by GitHub's security auditing tools).

Some changes fix vulnerabilities that are sensitive in nature, where exposure may pose a danger to Consensys, MetaMask, or community at large. These changes should be omitted entirely. (If in doubt, reach out to the MetaMask Security team.)

### Changed

A change should be filed under this category when it does not belong to any other category.

This could include:

- Changing the behavior of a public function or method ✅
- Adding or removing a "production" or peer dependency ✅
- Modifying a TypeScript type union or enum ✅
- Modifying an array, map, or other data structure which is exported directly ✅
- Adding a property to a TypeScript type ✅
- Removing a property from a TypeScript type ✅

In some cases, these kinds of changes should be marked as breaking (see ["Highlight breaking changes"](#higlight-breaking-changes) for the full list).

Notably, the Changed category does not include:

- Adding or removing a development dependency ❌
  - This should be excluded from the changelog, since it is only relevant for developers

## Highlight breaking changes

A change is "breaking" if it forces the consumer to take some kind of action after upgrading to prevent or handle one of the following:

- An error at install time (e.g., the consumer's package manager reports a Node version incompatibility)
- An error at compile time (e.g., a TypeScript type error)
- An error at runtime
- A surprising difference in behavior

A changelog entry which refers to such a change should be preceded with `**BREAKING:**` so that it is more visible to consumers.

Determining what is breaking is trickier than usual, particularly as it relates to TypeScript types. The following changes are always breaking:

- Changing the arity of a function or method (adding or removing positional arguments) ✅
- Adding a required option to the options bag of a function or method ✅
- Narrowing the TypeScript type of an argument in a function or method ✅
- Widening the return type of a function or method ✅
- Changing a function or method to throw an error ✅
- Adding a required property to a TypeScript type ✅
- Removing a property from a TypeScript type (even an optional one) ✅
- Narrowing the type of a property in a TypeScript object type ✅
- Upgrading a dependency to a version which causes any of the above ✅
- Adding a new peer dependency ✅
- Bumping the version of an existing peer dependency ✅
- Removing an export from the package ✅
- Bumping the minimum supported Node.js version ✅

The following changes may or may not be breaking, depending on whether types are intended to be ["user-constructable"](https://www.semver-ts.org/formal-spec/1-definitions.html):

- Adding an optional property to a TypeScript object type ❓
- Narrowing the type of a property in a TypeScript object type ❓
- Widening the TypeScript type of an argument in a function or method ❓
- Narrowing the return type of function or method ❓

### Read more

- ["Breaking Changes" in "Semantic Versioning for TypeScript Types"](https://www.semver-ts.org/formal-spec/2-breaking-changes.html)

## Omit non-consumer-facing changes

Since changelogs should be geared toward consumers, any other changes that do not have a material effect on the usable parts of a package should be omitted from the changelog.

This includes:

- Adding or removing a development dependency ❌
- Refactoring code ❌
- Adding new tests ❌
- Adding tooling, CI checks, or making other infrastructure changes that only benefit developers ❌

## Describe changes to the surface area of a project

It is tempting when leaving a changelog entry to simply reuse the message for the commit that introduced the change. Projects like [`release-please`](https://github.com/googleapis/release-please) or [`semantic-release`](https://github.com/semantic-release/semantic-release) have certainly popularized this practice. Due to character length requirements, however, commit messages can be rather cryptic, and so they are not as helpful in practice as they could be.

Using succinct language is good, but it is more important to describe the exact changes that have made to the project's API, CLI, or GUI. (For instance, if a method in a class has changed, mention both.) Including this level of detail helps consumers who are upgrading to a new version of a project understand how they can (or, in some cases, _must_) use the project going forward. It also assists consumers who are looking to compare and understand the capabilities between two versions of the same project in different consuming projects.

Be exact even when [linking out to a pull request](#include-links-to-pull-requests). Respect the reader's time and save them from needing to click through to the PR to obtain key information if it is possible to do so.

🚫 **Doesn't mention any changes to the API**

```markdown
- Add custom `getTransactions` ([#123](https://github.com/MetaMask/sample-project/pull/123))
```

✅ **Mentions parts of the API affected**

```markdown
- Make `fetch` argument for `fetchTokens` and `fetchTopAssets` optional, defaulting to a cached version ([#123](https://github.com/MetaMask/sample-project/pull/123))
```

It may be desirable to couch new changes in terms of features that they enable in consuming projects. If so, consider [describing the specific interface differences underneath the broad description](#use-a-nested-list-to-provide-context-or-details-about-a-change) (although be careful not to [group too many things together](#split-disparate-changes-from-the-same-pull-request-into-multiple-entries-if-necessary) so they can be categorized appropriately).

## 💡 Use a nested list to provide context or details about a change

Sometimes it is desirable to provide details for a change, such as context, purpose, consequences, or instructions for the consumer to follow, but it would be unwieldy to incorporate that information on the same line as the change itself.

In this situation, you can provide a broad description of the change and then give specifics in a nested list underneath it, similar in spirit to a commit message. For example:

🚫 **Too long**

```markdown
- Remove `trackMetaMetricsEvent` option from the NetworkController constructor, which was previously used in `upsertNetworkConfiguration` to create a MetaMetrics event when a new network was added, and should be replaced with a subscription to the `NetworkController:networkAdded` event ([#123](https://github.com/MetaMask/sample-project/pull/123))
```

✅ **The main gist has been preserved, and more details have been added underneath**

```markdown
- Remove `trackMetaMetricsEvent` option from the NetworkController constructor ([#123](https://github.com/MetaMask/sample-project/pull/123))
  - Previously, this was used in `upsertNetworkConfiguration` to create a MetaMetrics event when a new network was added. This can now be achieved by subscribing to the `NetworkController:networkAdded` event and creating the event inside of the event handler.
```

## Include links to pull requests

After [providing a description of changes to the surface area](#describe-changes-to-the-surface-area-of-the-project), add links to the pull requests that included the changes:

🚫 **Doesn't include any PR link**

```markdown
- Make `fetch` argument for `fetchTokens` and `fetchTopAssets` optional, defaulting to a cached version
```

✅ **Includes a PR link**

```markdown
- Make `fetch` argument for `fetchTokens` and `fetchTopAssets` optional, defaulting to a cached version ([#123](https://github.com/MetaMask/sample-project/pull/123))
```

✅ **Includes more than one PR link**

```markdown
- Make `fetch` argument for `fetchTokens` and `fetchTopAssets` optional, defaulting to a cached version ([#111](https://github.com/MetaMask/sample-project/pull/111), [#222](https://github.com/MetaMask/sample-project/pull/222))
```

## Combine like changes from multiple pull requests into a single entry if necessary

There is no requirement that each changelog entry map to exactly one commit. A changelog is not the same thing as a commit history, and sometimes it makes more sense to group together similar changes across multiple commits into one entry.

When doing so, make sure to include all relevant pull request links on the same line:

🚫 **These are separate entries that could really be one**

```markdown
- Bump `@metamask/utils` from `^0.9.7` to `^1.0.0` ([#111](https://github.com/MetaMask/sample-project/pull/111))
- Bump `@metamask/utils` from `^1.0.0` to `^2.0.0` ([#222](https://github.com/MetaMask/sample-project/pull/222))
```

✅ **They are now combined**

```markdown
- Bump `@metamask/utils` from `^0.9.7` to `^2.0.0` ([#111](https://github.com/MetaMask/sample-project/pull/111), [#222](https://github.com/MetaMask/sample-project/pull/222))
```

## Split disparate changes from the same pull request into multiple entries if necessary

Often, a commit contains distinct changes across different areas of the codebase. Although it is tempting to group changes by "feature", it can be easier for consumers looking for specific changes if specific changes to the surface area of the project are listed separately.

🚫 **This is one entry, but really should be several**

```markdown
### Changed

- Replace deprecated legacy feature flag implementation with new implementation ([#111](https://github.com/MetaMask/sample-project/pull/111))
  - Remove `LegacyFeatureFlag` type
  - Add `FeatureFlag` type
  - Add new method `fetchFeatureFlags`
  - Remove `fetchLegacyFeatureFlags`
```

✅ **The changes are now listed separately and divided into multiple categories**

```markdown
### Added

- Add `FeatureFlag` type ([#111](https://github.com/MetaMask/sample-project/pull/111))
- Add new method `fetchFeatureFlags` ([#111](https://github.com/MetaMask/sample-project/pull/111))

### Removed

- Remove deprecated `LegacyFeatureFlag` ([#111](https://github.com/MetaMask/sample-project/pull/111))
- Remove deprecated `fetchLegacyFeatureFlags` method ([#111](https://github.com/MetaMask/sample-project/pull/111))
```

If it is desirable to preserve context for the new changes, [use a nested list](#use-a-nested-list-to-provide-context-or-details-about-a-change) (but make sure to only group together entries within the same category):

✅ **Context is provided using a nested list**

```markdown
### Added

- Add new implementation for feature flags ([#111](https://github.com/MetaMask/sample-project/pull/111))
  - Add `FeatureFlag` type
  - Add new method `fetchFeatureFlags`

### Removed

- Remove legacy feature flag implementation ([#111](https://github.com/MetaMask/sample-project/pull/111))
  - Remove deprecated `LegacyFeatureFlag`
  - Remove deprecated `fetchLegacyFeatureFlags` method
```

## Remove reverted changes

Sometimes changes show up in the commit history and are then later reverted. Since consumers will never see those changes, there is no need to list them in the changelog:

🚫 **All commits show up in the changelog, even the revert commit**

```markdown
- Upgrade `lodash` to `^9.0.1` ([#111](https://github.com/MetaMask/sample-project/pull/111))
- Update `labelIssues` so that it will also add labels from some other project ([#111](https://github.com/MetaMask/sample-project/pull/111))
- revert: Upgrade `lodash` to `^9.0.1` ([#111](https://github.com/MetaMask/sample-project/pull/111))
```

✅ **The changelog is now cleaned**

```markdown
- Update `labelIssues` so that it will also add labels from some other project ([#111](https://github.com/MetaMask/sample-project/pull/111))
```
