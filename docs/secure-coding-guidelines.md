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

  For example, a URL for a website or API would typically have a scheme of `https`.

- Avoid dynamic code execution with untrusted data

  Dynamic code execution of untrusted data can allow for injection attacks. Prevent this by avoiding dynamic code execution completely where possible, but especially when the code being run was derived from untrusted data.

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
  - **Non-sensitive** data (e.g. publicly available, transactions, app theme settings, etc…)
  - **Sensitive** data (e.g. passwords, keys)

#### Data Security

- Encrypt **sensitive** data prior to persisting
- Limit the amount of time **sensitive** data is decrypted

#### Encryption

- **Sensitive** data should be secured at rest and in-transit using standard encryption protocols (e.g. cryptography libraries, OS level key management systems, https, etc…)
- Cryptography of **sensitive** data should meet a minimum standard established by the security team during a threat assessment and annual reviews

#### Passwords

- Passwords used to decrypt content should adhere to security team recommendations during a threat assessment and annual reviews.

### Logging & Error Handling

#### Logging

- Secrets and **sensitive** data should not be logged
- Console logging should be removed from production code using linters

#### Error Handling

- All errors should be handled
- Errors should not leak data and be made more generic. This is especially important for code which handles data classified as **sensitive**.
- Production versions of applications shall have logging disabled

### Third Party Integrations & Applications

#### Authentication and Authorization

- Implement access controls and authentication mechanisms for accessing external data
- Store and manage API keys securely when accessing external sources
- API keys used in client applications shall have a reduced scope adhering to Principle of Least Privilege
- Do not bundle API tokens in client side applications unless you intend for them to be publicly accessible

#### Integration/Application Integrity

- Monitor integrations/applications for changes and security issues.

#### Permissions

- All applications/dapps/snaps shall adhere to the following to Principle of Least Privilege with a default state of allowing no permissions
- Users shall provide consent for required permissions of applications/dapps/snaps

### Dependency Management

#### Dependency Integrity

- Use Dependabot to keep all dependencies up to date and to automate checks for updates
- Pin dependency versions or use a yarn.lock file

#### Avoid Deprecated and Unmaintained Packages

- Using Socket.dev to check the health and maintenance status of dependencies and seek alternatives when necessary

#### Review Dependency Licenses

- Ensure dependencies' licenses align with your project's requirements

#### Reduce Dependencies and Keep Them Minimal

- Minimize dependencies to reduce potential vulnerabilities

#### Audit and Monitor Dependencies

- Projects should use the npm audit, Dependabot and Socket.dev tooling for periodic automated auditing of dependencies

#### LavaMoat

- LavaMoat allow-scripts should be enabled on all projects
- LavaMoat runtime should be enabled on all projects

### Operations & Infrastructure

#### Project secrets

- Project secrets should be generated and shared using a password management system (1Password) and access to that system should follow the Principle of Least Authority
- Build systems needing access to project secrets should store secrets in specific secret storage and obscured once placed into system

#### Application Integrity

- Application should be distributed through approved store portals (GitHub, App Store, Play Store, Firefox Store, Chrome Store)
- Each distribution channel shall have provide release verifiability (certificate signing approval, hash verification)

#### Auditability

- All systems supporting the application shall have capability to audit modifications to the system
