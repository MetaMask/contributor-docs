# Ideal Package Structure

## Outline

### Source and Build Files

- Source files should be placed in a directory named `src`.
- Build files should be placed in a directory named `dist`.
- It is important to ensure that the source files are organized in a logical and modular way, with each file containing a single module or class.
- The build files should be generated automatically using a build tool such as Webpack or Rollup, and should not be committed to the repository.

### Tests and Test Helpers

- Tests should be placed in a directory named `test`.
- Test helpers should be placed in a directory named `test-helpers`.
- It is important to write comprehensive tests for all modules and classes, and to ensure that the tests are run automatically as part of the build process.
- Test helpers should be used to provide common functionality that is needed by multiple tests, such as setting up test data or mocking external dependencies.

### Types

- Types should be included in the package by adding a `types` field to the `package.json` file.
- Types should be written using TypeScript or Flow, and should be kept up-to-date with the latest version of the package.
- It is important to ensure that the types are accurate and comprehensive, and that they cover all public APIs of the package.

### Bash Scripts

- Bash scripts used for development tools or CI should be placed in a directory named `scripts`.
- It is important to ensure that the scripts are well-documented and easy to use, and that they are kept up-to-date with the latest version of the package.
- Bash scripts should be used sparingly, and only for tasks that cannot be easily accomplished using existing build tools or package scripts.

### Contributor Documentation

- Contributor documentation should be placed in a directory named `contributing`.
- Contributor documentation should include information on how to contribute to the package, including guidelines for submitting pull requests and reporting issues.
- It is important to ensure that the contributor documentation is up-to-date and accurate, and that it is easy to understand for first-time contributors.

### README

- The README should include a brief description of the package, installation instructions, usage instructions, and a list of contributors.
- It is important to ensure that the README is well-written and easy to understand, and that it provides all the information that a user would need to get started with the package.
- The README should be kept up-to-date with the latest version of the package, and should be reviewed periodically to ensure that it is still accurate and relevant.

### Package.json Fields/Metadata

- The `package.json` file should include the following fields:
  - `name`: The name of the package.
  - `version`: The version of the package.
  - `description`: A brief description of the package.
  - `main`: The entry point for the package.
  - `scripts`: A list of scripts that can be run with `npm run`.
  - `repository`: The URL of the repository.
  - `keywords`: A list of keywords that describe the package.
  - `author`: The name and email address of the package author.
  - `license`: The license under which the package is released.
  - `bugs`: The URL where issues can be reported.
  - `homepage`: The URL of the package's homepage.
- It is important to ensure that all fields are accurate and up-to-date, and that they provide all the information that a user would need to understand and use the package.
- The `name` field should be unique and descriptive, and should follow the naming conventions for npm packages.
- The `version` field should follow the semantic versioning conventions.
- The `description` field should provide a brief overview of the package's functionality.
- The `main` field should point to the entry point for the package.
- The `scripts` field should include all the scripts that are needed to build, test, and run the package.
- The `repository` field should point to the URL of the repository.
- The `keywords` field should include a list of keywords that describe the package.
- The `author` field should include the name and email address of the package author.
- The `license` field should specify the license under which the package is released.
- The `bugs` field should point to the URL where issues can be reported.
- The `homepage` field should point to the URL of the package's homepage.

### License

- The license should be placed in a file named `LICENSE`.
- It is recommended to use a standard open-source license, such as the MIT License or Apache License.
- Before releasing a package, it is important to get approval from the legal team to ensure that the license is appropriate for the project.
- It is important to ensure that the license is included in all distributions of the package, and that it is clearly visible and easy to understand.

### Generated API Documentation

- Generated API documentation should be placed in a directory named `docs`.
- The documentation should be hosted on `metamask.github.io/<package-name>/<latest|staging|version>`.
- It is important to ensure that the API documentation is accurate and up-to-date, and that it covers all public APIs of the package.
- The documentation should be generated automatically using a tool such as JSDoc or TypeDoc, and should be reviewed periodically to ensure that it is still accurate and relevant.

### Configuration Files

- Configuration files should be placed in a directory named `config`.
- Configuration files should be used to store configuration data that is needed by the package, such as API keys or database credentials.
- It is important to ensure that the configuration files are well-documented and easy to use, and that they are kept up-to-date with the latest version of the package.

### Keeping Packages in Sync with the Module Template

- To keep packages in sync with the module template, use the `@metamask/template-sync` tool.
- It is important to ensure that the package is kept up-to-date with the latest version of the module template, and that any changes to the template are incorporated into the package as soon as possible.
- It is also important to ensure that any customizations to the package are not overwritten by updates to the module template, and that any conflicts are resolved in a timely manner.
