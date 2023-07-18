# Secure Development Lifecycle Policy

## Purpose
The secure developmet lifecycle describes the approach that is used to define, develop, deliver and maintain software. 

## Scope
This lifecycle is intended to be used by software teams including product managers, designers, developers and quality assurance to develop secure software products.

## Policy
This lifecycle integrates secruity into all aspects of application software development which is comprized of seven phases. 

1. Requirements Gathering and Planning:
   - Identify feature scope, goals, and resources required for both development and security testing.
   - Define functional and security requirements for the feature.
      - Consider security aspects such as (//TODO add ref) core engineering principles, authentication, data encryption and secure communication protocols.

2. Design:
   - Create a feature architecture that incorporates security controls identified. 
   - Perform threat modeling to identify potential risks and design appropriate countermeasures.
   - Document design decisions related to security controls and their implementation.

3. Development:
   - Follow coding standards and secure coding practices (//TODO add ref).
   - Implement feature and security controls according to the design.
   - Conduct code reviews to identify and fix issues.
   - Feature passes automated security testing checks in the development process.

4. Testing:
   - Perform functional testing to ensure the application meets the desired requirements.
   - Conduct security testing (e.g. static code analysis, dynamic scanning, and penetration testing).
   - Address any identified issues or vulnerabilities and validate fixes.
   - Document testing and results perfomed on feature. 

5. Deployment:
   - Prepare the application for deployment, considering secure configuration and hardening measures.
   - Utilize secure deployment practices, such as code signing and secure distribution channels.
   - Implement mechanisms for secure updates and patches.

6. Maintenance and Support:
   - Establish an incident response plan to handle security breaches or incidents. (//TODO add ref to security team response plan)
   - Monitor for issues and security vulnerabilities and apply patches or updates as needed (//TODO add ref to bug triage).
   - Provide ongoing support, including user education on security best practices.

7. Post-Deployment:
   - Collect and analyze feedback from users to identify potential security issues.
   - Incorporate security enhancements and bug fixes into future releases.
   - Continuously evaluate and improve the security posture of the application.