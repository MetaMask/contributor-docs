# Engineering Principles

Engineering principles define the core tenets and beliefs that the engineering team and all contributors are expected to keep front of mind. These tenets are meant to reflect the values of the organization.

## Core Engineering Values

These are the qualities that we strive for in our software. (These values aren't ranked, but are all of equal importance.)

- Security
- Stability
- Privacy
- Maintainability
- Usability
- Transparency

## Principles

### The Principle of Least Authority

Provide application code the absolute minimal amount of authority required to perform its essential functions. Wherever appropriate, break down large methods with broad authority into smaller functions with minimal authority. Minimizing what a method _can_ do reduces the scope of what could possibly go wrong. When something does go wrong, small, isolated methods, written with the least amount of authority required, are easier to troubleshoot.

### The Principle of Incremental Change

To minimize the risk to the stability and security of the application, changes to the code should be incremental and as small as possible. Large changes to the codebase become difficult to review effectively and make the commit history less itemized. Furthermore, incremental change promotes better test coverage and makes it easier to require tests to cover the introduced changes.

### The Principle of Incremental Decentralization

We are trying to build a decentralized future and we must make choices that move us directionally towards those goals. This means that users should be able to swap centralized services out for decentralized alternatives. Additionally, when we build features that rely upon centralized services we must build them in such a way where the experience isn't completely broken when the service is disrupted.

### The Principle of Self-Sovereign Identity

We believe in the right of our users to own their own identity and data. We must go to great lengths to protect that right. Where we have features or services that might infringe upon that right, we must treat those features as opt-in, not opt-out. When we do build features that make user data known to us, we must not collect any personally identifiable information, and any data that can be tied to any chain must be collected in an anonymity-protecting way. This principle must be extended to cover all third-party code as well.

### The Principle of Auditability

As we are committed to providing a high level of security and privacy within our product, we must also maintain an auditable, publicly accessible, and transparent codebase. We mustn't ask our users to trust our products implicitly, but instead, write code that explicitly illustrates our values. In order for our codebase to be easily auditable, our code must have the following qualities:

1. Readability - Our code should be easily read, and be free of obfuscation or subterfuge.
2. Consistency - Our code should follow consistent standards to assist auditors in tracing code paths across multiple repositories or code from varying authors.
3. Usability - Our code must maintain a standard of high quality and usability, and reflect the qualities listed in the values section above.
4. Traceability - Our code should have a public record of changes made, with details about why the change was made.

### The Principle of "Be Methodical and Fix Things"

We should prioritize fixing bugs and improving stability over creating new features whenever possible. All code generates new bugs and builds technical debt, this is inevitable. However, if we focus on building our features with stability in mind and throw out the "Move Fast and Break Things" mentality of most web 2.0 applications we can minimize the amount of bugs we introduce. Furthermore, whenever bugs are known to exist we should expend the necessary energy to remedy them.

### The Principle of Reliability

We provide a plethora of features in our applications that users expect to be supported, and that they rely upon for their day-to-day activities. We must handle feature deprecation gracefully, and inform users well in advance that a feature is being sunsetted or replaced. If we remove features or experiences from our product without warning it will alienate our users and cause them to seek out other solutions.

We also provide external facing APIs that app developers utilize to build their products. Every change we make to these APIs must be analyzed for potentially breaking functionality of the apps in our ecosystem. When we make breaking changes we are creating work for countless other organizations and individuals, and only by their good graces will they make the requested changes. We must view these breaking changes as a potential reset of our momentum within the ecosystem. Breaking changes should only be made for critical reasons of security or stability that cannot be made by any other means.

When removing a feature or making a breaking change to an API is necessary, then a generous migration period must be planned and ecosystem or user outreach must occur to ensure a smooth transition.
