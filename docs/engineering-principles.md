# Engineering Principles
Engineering principles define the core tenets and beliefs that the engineering team and all contributors are expected to keep front of mind. These tenets are meant to reflect the values of the organization as well as the philosophy upon which the software was founded. Principles typically come in the form of punchy one line statements, but a great deal of depth can be written on each one to further inform of the intent of the statement.

## Core Engineering Values
These one word statements are used to not only capture the qualities we strive for in our software, but also prioritizes those values. Keep in mind that being the last value in this list does not imply it is of no importance, it is still more important than all of the other qualities not mentioned here. 
1. Security
2. Stability
3. Safety
4. Privacy
5. Maintainability
6. Usability


## Principles
### The Principle of Least Authority

Provide application code the absolute minimal amount of authority required to perform it's essential functions. Wherever appropriate, break down large methods with broad authority into smaller functions with minimal authority. Minimizing what a method *can* do reduces the scope of what could possibly go wrong. When something does go wrong, small isolated methods, written with the least amount of authority required, are easier to troubleshoot.

### The Principle of Core Purity

Whenever possible prefer small, pure functions. Pure functions are those that given the same inputs will always return the same output, and that have no side effects. Where side effects must occur, keep the core of the application code pure and move imperative methods to the outer edges of the application layer.


### The Principle of Incremental Change

To minimize the risk to the stability and security of the application, changes to the code should be incremental and as small as possible. Large changes to the codebase become difficult to review effectively and make the commit history less itemized. Furthermore, incremental change promotes better test coverage and makes it easier to require tests to cover the introduced changes.

### The Principle of Incremental Decentralization

We may sometimes ship with centralized services by default, but if we must, users should be able to opt out of any of these during onboarding, ideally easily swapping them out for decentralized alternatives. When we build features that rely upon centralized services we must build them in such a way where the experience isn't completely broken when the service is disrupted. We are trying to build a decentralized future and we must make choices that move us directionally towards those goals.

### The Principle of Self-Soverign Identity

We believe in the right of our users to own their own identity and data. We must go to great lengths to protect that right. Where we have features or services that might infringe upon that right we must treat those features as opt in, not opt out. When we do build features that make user data known to us we must not collect any personally identifiable information, and any data that can be tied to any chain must be collected in an anonymity protecting way. This principle must be extended to cover all third party code as well.

### The Principle of Auditability

All code contributions must be reviewed thoroughly for code quality, usability and adherence to these principles. Code contributions must be kept as small as possible (see Principle of Incremental Change) to promote auditability. Avoid long running feature branches because they are not auditable, and pose a risk to the stability and security of the product.


### The Principle of "Be Methodical and Fix Things"

We should prioritize fixing bugs and improving stability over creating new features whenever possible. All code generates new bugs and builds technical debt, this is inevitable. However, if we focus on building our features with stability in mind and throw out the "Move Fast and Break Things" mentality of most web 2.0 applications we can minimize the amount of bugs we introduce. Furthermore, whenever bugs are known to exist we should expend the necessary energy to remedy them. 

### The Principle of Ecosystem Momentum

We provide external facing APIs that developers rely upon to build out their products. Every change we make to those APIs must be analyzed for potentially breaking functionality of the apps in our ecosystem. When we make breaking changes we are creating work for countless other organizations and individuals, and only by their good graces will they make the requested changes. We must view these breaking changes as a potential reset of our momentum within the ecosystem. Breaking changes should only be made for critical reasons of seucrity or stability that cannot be made by any other means. Furthermore, a migration period of no less than six weeks must be planned and ecosystem outreach must occur to ensure a smooth transition.

