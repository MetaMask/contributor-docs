# Secure Development Lifecycle Policy

## Purpose
The secure development lifecycle describes the approach that is used to define, develop, deliver and maintain software. 

## Scope
This lifecycle is intended to be used by software teams including product managers, designers, developers and quality assurance to develop secure software products.

## Policy
This lifecycle integrates security into all aspects of application software development which is comprised of six phases. 

1. Requirements Gathering and Planning:
  - Identify the scope, goals, and necessary resources for development
  - Define functional and security requirements for the feature 
    - Considering core engineering principles (link)
    - Security controls such as authentication, data encryption, and secure communication protocols

2. Design:
- Create a feature description that incorporates identified security controls and meets definition of ready (link)
- Perform threat modeling to identify potential risks and design appropriate countermeasures
- Document design decisions, security controls, implementation and verification

3. Development:
- Adhere to coding standards and secure coding practices (link)
- Implement features and security controls according to the design
- Conduct code reviews to identify and address issues
- Ensure automated code quality and security testing checks are passing prior to testing

3. Testing:
- Perform functional testing to validate the application meets desired requirements
- Conduct security testing identified during the planning phase (e.g. static code analysis, dynamic scanning, and penetration testing)
- Address identified issues and vulnerabilities, and validate the effectiveness of fixes
- Document testing activities and results for the feature

4. Deployment:
- Prepare the application for deployment, considering secure configuration and hardening measures
- Utilize secure deployment practices, such as code signing and secure distribution channels
- Implement mechanisms for secure updates and patches to ensure ongoing security

5. Maintenance and Support:
- In the event of security breach or incident, follow established incident response plan (link)
- Monitor for issues and security vulnerabilities, and apply patches or updates according to response plan (link)

6. Post-Deployment:
- Collect and analyze user feedback to identify potential issues or security vulnerabilities
- Incorporate security enhancements and bug fixes into future releases
- Continuously evaluate and improve the security posture of the application