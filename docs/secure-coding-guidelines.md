# Secure Coding Guidelines

## Purpose

The secure coding guidelines are a reference to develop, deliver and maintain software in a secure manner.

## Scope

These guidelines apply to developers building client based applications at MetaMask.

## References

The guidelines in this policy were gathered primarily from the [OWASP Top 10](https://owasp.org/www-project-top-ten/) and the [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/). We have included just the guidelines most relevant to MetaMask client applications.

## Guidelines

### 1 Input from Users and Third Party Services

#### 1.1 Data Validation and Sanitization

1. Always validate and sanitize user inputs and reject any input that does not pass validation, such as:
   1. Checking the type is what we expect: (e.g. is a javascript object being passed in when a string is expected)
   2. Checking if the value is what within a range we expect: (e.g. is a negative number being provided when a positive one is expected)
   3. Checking if the format is what we expect: (e.g. do we expect a `0x` prefixed hex string but an alphanumeric string is provided instead)
   4. Checking if a value matches an allowed set of constants (e.g. is the value one of the few possible numbers we expect)
2. Validate and sanitize data from external sources before rendering it in the application's UI
3. Preference should be given to libraries and frameworks that support input validation
4. Avoid dynamic code execution with untrusted data to prevent injection attacks

#### 1.2 Content Security

1. Verify the content type of external data to ensure it matches expectations
2. Enforce proper type checking, file type validation for file/media uploads and rendering 3rd party content
3. Implement Content Security Policies (CSP) to mitigate XSS attacks when rendering external data

#### 1.3 Data Serialization

1. Recommend using pure data formats such as CSV, JSON for serialization to avoid deserialization attacks

### 2 Data Persistence & Encryption

#### 2.1 Data Classification

1. Classify the data in your application into at least two groups:
2. **Non-sensitive** data (e.g. publicly available, transactions, app theme settings, etc…)
3. **Sensitive** data (e.g. passwords, keys)

#### 2.2 Data Security

1. Encrypt **sensitive** data prior to persisting
2. Limit the amount of time **sensitive** data is decrypted
3. Do not bundle API tokens in client side applications unless you intend for them to be publicly accessible

#### 2.3 Encryption

1. **Sensitive** data should be secured at rest and in-transit using standard encryption protocols (e.g. cryptography libraries, OS level key management systems, https, etc…)
2. Cryptography of **sensitive** data should meet a minimum standard established by the security team during a threat assessment and annual reviews

#### 2.4 Passwords

1. Passwords used to decrypt content should adhere to security team recommendations during a threat assessment and annual reviews.

### 3 Logging & Error Handling

#### 3.1 Logging

1. Secrets and **sensitive** data should not be logged
2. Console logging should be removed from production code using linters

#### 3.2 Error Handling

1. All errors should be handled
2. Errors should not leak data and be made more generic. This is especially important for code which handles data classified as **sensitive**.
3. Production versions of applications shall have logging disabled

### 4 3rd Party Integrations & Applications

#### 4.1 Authentication and Authorization

1. Implement access controls and authentication mechanisms for accessing external data
2. Store and manage API keys securely when accessing external sources
3. API keys used in client applications shall have a reduced scope adhering to Principle of Least Privilege
4. If an API key is not intended to be publicly accessible, do not use it in a client application

#### 4.2 Integration/Application Integrity

1. Monitor integrations/applications for changes and security issues.

#### 4.3 Permissions

1. All applications/dapps/snaps shall adhere to the following to Principle of Least Privilege with a default state of allowing no permissions
2. Users shall provide consent for required permissions of applications/dapps/snaps

### 5 Dependency Management

#### 5.1 Dependency Integrity

1. Use Dependabot to keep all dependencies up to date and to automate checks for updates
2. Pin dependency versions or use a yarn.lock file

#### 5.2 Avoid Deprecated and Unmaintained Packages

1. Using Socket.dev to check the health and maintenance status of dependencies and seek alternatives when necessary

#### 5.3 Review Dependency Licenses

1. Ensure dependencies' licenses align with your project's requirements

#### 5.4 Reduce Dependencies and Keep Them Minimal

1. Minimize dependencies to reduce potential vulnerabilities

#### 5.5 Audit and Monitor Dependencies

1. Projects should use the npm audit, Dependabot and Socket.dev tooling for periodic automated auditing of dependencies

#### 5.6 LavaMoat

1. LavaMoat allow-scripts should be enabled on all projects
2. LavaMoat runtime should be enabled on all projects

### 6 Operations & Infrastructure

#### 6.1 Project secrets

1. Project secrets should be generated and shared using a password management system (1Password) and access to that system should follow the Principle of Least Authority
2. Build systems needing access to project secrets should store secrets in specific secret storage and obscured once placed into system

#### 6.2 Application Integrity

1. Application should be distributed through approved store portals (GitHub, App Store, Play Store, Firefox Store, Chrome Store)
2. Each distribution channel shall have provide release verifiability (certificate signing approval, hash verification)

#### 6.3 Auditability

1. All systems supporting the application shall have capability to audit modifications to the system
