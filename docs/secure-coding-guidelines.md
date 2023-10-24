# Secure Coding Guidelines

## Purpose

The secure coding guidelines are a reference to develop, deliver and maintain software in a secure manner.

## Scope

These guidelines apply to developers building client based applications at MetaMask.

## References

The guidelines in this policy were gathered from concepts on the [OWASP Top 10 Site](https://owasp.org/www-project-top-ten/) and modified to meet the client applications being developed at MetaMask.

## Guidelines

This document identifies six categories developers should reference when working on client based applications.

1. User Input:

   1. Processing User Inputs:
      1. Always validate and sanitize user inputs
      2. Preference should be given to libraries and frameworks that support input validation
   2. Data Serialization:
      1. Recommend using pure data formats such as JSON for serialization to avoid deserialization attacks

2. Data Persistence & Encryption:

   1. Data Classification:
      1. Classify the data in your application into at least two groups:
      1. Non-sensitive data (e.g. publicly available, transactions, tokens, etc…)
      1. Sensitive data (e.g. passwords, keys)
   2. Data Security:
      1. Encrypt sensitive data prior to persisting
      2. Limit the amount of time sensitive data is decrypted
   3. Encryption:
      1. Sensitive data should be secured at rest and in-transit using standard encryption protocols (e.g. cryptography libraries, OS level key management systems, https, etc…)
      2. Cryptography of sensitive data should meet a minimum standard established by the security team during a threat assessment and annual reviews
   4. Passwords:
      1. Passwords used to decrypt content should adhere to security team recommendations during a threat assessment and annual reviews.

3. Logging & Error Handling:

   1. Logging:
      1. Secrets and sensitive data should not be logged
      2. Console logging should be removed from production code using linters
   2. Error Handling:
      1. All errors should be handled
      2. Errors should not leak data and be made more generic
      3. Production versions of applications shall have logging disabled

4. 3rd Party Data & Applications:

   1. Data Validation and Sanitization:
      1. Validate and sanitize data from external sources before rendering it in the application's UI
      2. Avoid dynamic code execution with untrusted data to prevent injection attacks
   2. Content Security:
      1. Verify the content type of external data to ensure it matches expectations
      2. Enforce proper type checking, file type validation for file/media uploads and rendering 3rd party content
      3. Implement Content Security Policies (CSP) to mitigate XSS attacks when rendering external data
   3. Authentication and Authorization:
      1. Implement access controls and authentication mechanisms for accessing external data
      2. Store and manage API keys securely when accessing external sources
      3. API keys used in client applications shall have a reduced scope adhering to Principle of Least Privilege
   4. Data Integrity:
      1. Monitor external data sources for changes and security issues.
   5. Permissions:
      1. All applications/dapps/snaps shall adhere to the following to Principle of Least Privilege with a default state of allowing no permissions
      2. Users shall provide consent for required permissions of applications/dapps/snaps

5. Dependency Management:

   1. Dependency Integrity:
      1. Keep all dependencies up to date, using tools to automate checks for updates
      2. Pin dependency versions or use a yarn.lock file
   2. Avoid Deprecated and Unmaintained Packages:
      1. Using Socket.dev to check the health and maintenance status of dependencies and seek alternatives when necessary
   3. Review Dependency Licenses:
      1. Ensure dependencies' licenses align with your project's requirements
   4. Reduce Dependencies and Keep Them Minimal:
      1. Minimize dependencies to reduce potential vulnerabilities
   5. Audit and Monitor Dependencies:
      1. Projects should use the npm audit, Dependabot and Socket.dev tooling for periodic automated auditing of dependencies
   6. LavaMoat:
      1. LavaMoat allow-scripts should be enabled on all projects
      2. LavaMoat runtime should be enabled on all projects

6. Operations & Infrastructure:
   1. Project secrets:
      1. Project secrets should be generated and shared using a password management system (1Password) and access to that system should follow the Principle of Least Authority
      2. Build systems needing access to project secrets should store secrets in specific secret storage and obscured once placed into system
   2. Application Integrity:
      1. Application should be distributed through approved store portals (GitHub, App Store, Play Store, Firefox Store, Chrome Store)
      2. Each distribution channel shall have provide release verifiability (certificate signing approval, hash verification)
   3. Auditability:
      1. All systems supporting the application shall have capability to audit modifications to the system
