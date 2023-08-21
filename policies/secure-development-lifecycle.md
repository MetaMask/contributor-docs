# Secure Development Lifecycle Policy

## Purpose

The secure development lifecycle describes the approach that is used to define, develop, deliver and maintain software.

## Scope

This lifecycle is intended to be used by software teams including product managers, designers, developers and quality assurance to develop secure software products. This policy describes the lifecycle of feature development.

## Policy

This lifecycle integrates security into all aspects of application software development which is comprised of six phases.

1. Requirements Gathering and Planning:

- Identify the scope, goals, and necessary resources for development
- Define functional and security requirements for the feature
  - Considering [core engineering principles](https://github.com/MetaMask/contributor-docs/blob/main/policies/engineering-principles.md)
  - Security controls such as authentication, data encryption, and secure communication protocols
- Inform the marketing team and customer support of the projected timelines, features, and any potential market impacts

2. Design:

- Create a feature description that incorporates identified security controls and meets definition of ready
- Perform threat modeling to identify potential risks and design appropriate countermeasures (e.g. [4 question framework](https://github.com/adamshostack/4QuestionFrame))
- Document design decisions, security controls, implementation and verification

3. Development:

- Adhere to [coding standards and secure coding practices](https://github.com/MetaMask/contributor-docs/tree/main/guides)
- Implement features and security controls according to the design
- Conduct code review for all changes
- Ensure automated code quality and security testing checks are passing

4. Testing:

- Perform testing to validate the application meets desired requirements. The testing methodology is set by project specific guidelines.
- Conduct security testing identified during the planning phase (e.g. static code analysis, dynamic scanning, and penetration testing)
- Address identified issues and vulnerabilities, and validate the effectiveness of fixes
- Document testing activities and results for the feature

5. Release and Monitoring:

- Prepare the application for deployment, considering secure configuration and hardening measures
  - Utilize secure deployment practices, such as code signing and secure distribution channels
  - Implement mechanisms for secure updates and patches to ensure ongoing security
- In the event of security breach or incident, follow established incident response plan
- Monitor for issues and security vulnerabilities, and apply patches or updates according to triage framework

6. Post-Release:

- Collect and analyze user feedback to identify potential issues or security vulnerabilities
- Incorporate security enhancements and bug fixes into future releases
- Continuously evaluate and improve the security posture of the application
