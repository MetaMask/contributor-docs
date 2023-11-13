# Secure Coding Guidelines

## Purpose

The secure coding guidelines are a reference to develop, deliver and maintain software in a secure manner.

## Scope

These guidelines apply to developers building client based applications at MetaMask.

## References

The guidelines in this policy were gathered primarily from the [OWASP Top 10](https://owasp.org/www-project-top-ten/) and the [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/). We have included just the guidelines most relevant to MetaMask client applications.

## Guidelines

### Input from Users and Third Party Services

#### Data Validation and Sanitization

- Validate and sanitize all user input, and all data from external sources

  For example:

  - Check that the type matches expectations
    - If a value should be a string, do not allow it to be an object
    - Validation libraries can be used for more complex types of data
  - Check that the value is is an allowed value or within the expected range
    - If only a fixed set of values are expected, ensure the value matches one of them
    - If a non-negative number is expected, do not allow the value to be negative
  - Check that the format matches expectations
    - If we expect a 0x-prefixed hexadecimal string, ensure that the 0x is present

- Encode data before output

  For example:

  - HTML-encode strings before they are included in the DOM
    - Some libraries do this automatically (e.g. React)
  - URL-encode data before including it in a URL

- When accepting a URI as input, ensure the scheme matches expectations

  For example, a URL for a website or API would typically have a scheme of `https`

- Avoid dynamic code execution with untrusted data

  Dynamic code execution of untrusted data can allow for injection attacks. If possible, avoid dynamic code execution completely. If you require dynamic code execution, consult with the security team on how to do this safely.

  Examples of dynamic code execution:

  - `eval`
  - `new Function`
  - Template processors (e.g. for HTML, SQL, etc.)
  - DOM
  - `javascript:` protocol

#### Content Security Policy

- Use a Content Security Policy (CSP) to mitigate XSS attacks when rendering external data

#### Data Serialization

- Use pure data formats (e.g. JSON, CSV) for serialization to avoid deserialization attacks

### Data Persistence & Encryption

#### Data Classification

- Classify the data in your application into at least two groups:
  - **Non-sensitive** data (e.g. publicly available, transactions, app theme settings, etc.)
  - **Sensitive** data (e.g. passwords, keys)

#### Data Security

- Encrypt **sensitive** data prior to persisting
- Limit access to **sensitive** data

#### Encryption

- **Sensitive** data should be secured at rest and in-transit using standard encryption protocols (e.g. cryptography libraries, OS level key management systems, https, etc.)
- Cryptography of **sensitive** data should meet a minimum standard established by the security team

#### Passwords

- Passwords used to decrypt content should adhere to security team recommendations

### Logging & Error Handling

#### Logging

- Secrets and **sensitive** data should not be logged

#### Error Handling

- Avoid including data directly in error messages, especially **sensitive** data

  For example, the error "Invalid hex data: \_\_\_\_" might indadvertantly leak a private key. Instead the error could be made more generic ("Invalid hex data"), or you could describe the problem without embedding the data ("Invalid hex data; missing 0x prefix").

### Third Party Integrations

#### Authentication and Authorization

- Do not bundle API tokens in client side applications unless you intend for them to be publicly accessible

#### Integration/Application Integrity

- Monitor integrations/applications for changes and security issues

### Third Party Applications

#### Permissions

- All applications/dapps/snaps shall adhere to the following to Principle of Least Privilege with a default state of allowing no permissions
- Users shall provide consent for required permissions of applications/dapps/snaps

### Dependency Management

#### Dependency Integrity

- Use a lockfile or pinned dependencies to maintain control over which version of each dependency is used

#### Avoid Deprecated and Unmaintained Packages

- Using Socket.dev to check the health and maintenance status of dependencies and seek alternatives when necessary

#### Reduce Dependencies and Keep Them Minimal

- Minimize dependencies to reduce potential vulnerabilities

#### Audit and Monitor Dependencies

- Monitor dependencies for security vulnerabilities and other problems
  - Periodically scan for security vulnerabilities (e.g. using tools like `npm audit`)
  - Update dependencies quickly when they have security vulnerabilities
  - Use Socket.dev to monitor dependencies for other noteworthy changes, such as maintainer changes, or the addition of install scripts or binary files
    - Use the following etiquette when addressing Socket.dev warnings:
      - Investigate and address all warnings before merging a PR
      - Avoid using the `ignore-all` bot command, instead ignoring each warning one at a time
      - If you've investigated a warning and found that it's not indicative of a malicious dependency, ignore it with a bot comment and explain your investigation with a short comment
      - Contact the security team if you're unsure how to investigate something, or if you'd like to disable a warning category

#### LavaMoat (JavaScript projects only)

- LavaMoat `allow-scripts` should be enabled on all projects
  - This project relies upon install scripts being disabled in your package manager. This can be verified by adding the dependency `@lavamoat/preinstall-always-fail`, which will cause installation to fail if scripts are enabled.
  - `allow-scripts` acts as an allowlist for install scripts. Use the `allow-scripts` binary after installing dependencies to run install scripts on the allowlist.
    - On Yarn v3 projects, use the `yarn-plugin-allow-scripts` plugin to run allowed scripts automatically during install
    - On other projects, use an npm script called `setup` that will call `install` and `allow-scripts` in sequence
  - If you're unsure whether an install script is needed, leave it disabled
- LavaMoat runtime should be used for all Node.js build systems, and LavaMoat bundling tools should be used for all JavaScript client applications
  - The LavaMoat runtime helps protect against malicious code in dependencies by isolating dependencies and reducing their capabilities
  - The LavaMoat bundling tools will bundle client applications with a LavaMoat runtime and policy
  - Regularly review your LavaMoat policies for suspicious permissions
  - When the policy is updated, carefully review the diff to ensure that the capabilities granted to each dependency seem appropriate and legitimate
  - Consult with the LavaMoat team if you have any questions

### Operations & Infrastructure

#### Project secrets

- Store and manage project secrets securely
  - Each CI system should have a method of securely managing secrets
  - For secrets used directly by developers, use 1Password
- Limit access to project secrets, following the Principle of Least Authority
- Prevent contributors from committing secrets to the repository
  - GitHub has a "Secret scanning" setting that can help with this, and there are third-party tools like `2ms` (too many secrets) that can help as well

#### Application Integrity

- Application should be distributed through approved store portals (e.g. GitHub, App Store, Play Store, Firefox Store, Chrome Store)
- Each distribution channel shall have provide release verifiability (e.g. certificate signing approval, hash verification)
