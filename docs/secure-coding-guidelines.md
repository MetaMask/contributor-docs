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

- Always validate and sanitize user inputs and reject any input that does not pass validation, such as:
  1.  Checking the type is what we expect: (e.g. is a javascript object being passed in when a string is expected)
  2.  Checking if the value is what within a range we expect: (e.g. is a negative number being provided when a positive one is expected)
  3.  Checking if the format is what we expect: (e.g. do we expect a `0x` prefixed hex string but an alphanumeric string is provided instead)
  4.  Checking if a value matches an allowed set of constants (e.g. is the value one of the few possible numbers we expect)
- Validate and sanitize data from external sources before rendering it in the application's UI
- Preference should be given to libraries and frameworks that support input validation
- Avoid dynamic code execution with untrusted data to prevent injection attacks

#### Content Security

- Verify the content type of external data to ensure it matches expectations
- Enforce proper type checking, file type validation for file/media uploads and rendering third party content
- Implement Content Security Policies (CSP) to mitigate XSS attacks when rendering external data

#### Data Serialization

- Recommend using pure data formats such as CSV, JSON for serialization to avoid deserialization attacks

### Data Persistence & Encryption

#### Data Classification

- Classify the data in your application into at least two groups:
- **Non-sensitive** data (e.g. publicly available, transactions, app theme settings, etc…)
- **Sensitive** data (e.g. passwords, keys)

#### Data Security

- Encrypt **sensitive** data prior to persisting
- Limit the amount of time **sensitive** data is decrypted
- Do not bundle API tokens in client side applications unless you intend for them to be publicly accessible

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
- If an API key is not intended to be publicly accessible, do not use it in a client application

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
