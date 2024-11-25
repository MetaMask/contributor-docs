# Contributing to the MetaMask Platform

This is an overview of how to contribute to the MetaMask platform (which includes MetaMask extension, MetaMask mobile, and the set of libraries we use on both clients). Different teams within MetaMask work in different ways, but this document describes workflows, processes, and guidelines that are applicable to all work in MetaMask platform repositories.

## Communication

### Slack

We use Slack as our primary method of communication. Here are the team's "core" channels:

* `metamask-general` - General announcements and discussions
* `metamask-dev` - Engineering-related discussions not tied to a specific team/product.
* `metamask-extension` - MetaMask extension discussions
* `metamask-mobile` - MetaMask mobile discussions not related to engineering
* `metamask-mobile-dev` - MetaMask mobile discussions related to engineering
* `metamask-biz-ops` - Operational questions/discussions (e.g. getting access to systems, managing billing for third-party services, etc.)
* `metamask-security` - Discussions on security
* `metamask-release-updates` - Release updates for MetaMask extension and mobile

All contributors are encouraged to ask questions in public channels, so that it's easier for others to contribute and benefit from the discussion. Don't be afraid of asking a "dumb question"; if you are thinking it, the chances are that others are too.

### Calendars

* "MetaMask Sync", for team-wide meetings
* "MetaMask Optional", for optional meetings. Feel free to attend any meeting on this calendar, even if you're not directly invited.

## Software Development Workflows

Here are the most common software development processes/workflows. For a more complete overview, see the [SDLC](./sdlc.md) documentation.

### Planning / Design

#### Issue tracking

We use GitHub Issues for MetaMask Platform issue tracking. Some teams use Jira as well, but all bugs that impact MetaMask platform are tracked on GitHub.

Each repository has issue templates that ensure the appropriate information is captured. Please complete the issue template to the best of your ability when creating an issue.

Most often we track issues in the repository they most directly impact (e.g. an issue about a MetaMask extension feature or bug would go in `metamask-extension`). For private issues (e.g. security-related), or for cross-repository issues, we also use `metamask-planning` and `mobile-planning`.

We organize our issues using labels. Every triaged issue should have a label describing the type of the issue (enhancement/feature or bug), the team(s) responsible for it. Additionally, bug issues are given additional labels describing the severity of the bug, where it was first introduced, and where it was fixed.

#### UI designs

We use Figma for UI designs. Features involving new UI designs will include a Figma link.

> [!TIP]
> Figma links are not accessible by external contributors. We can share screenshots upon request.

#### Product Decisions

Teams keep track of their planned features and changes in different ways, but we all use the same process for making decisions that require more in-depth discussion, or consultation with a wide number of stakeholders (e.g. more than one team). This is the [RAPID decision making framework](https://docs.google.com/document/d/1Besg-CXyBavfLGjO6d5Uyd1em7PfmSc2l4M6jOYMBTc/edit?tab=t.0#heading=h.arsqaq3k7byh).

For public-facing developer API changes, we use [MetaMask Improvement Proposals](https://github.com/MetaMask/metamask-improvement-proposals/) or [Snaps Improvement Proposals](https://github.com/MetaMask/SIPs) (for the Wallet API and Snaps API respectively).

#### Technical Decisions

We use Architectural Decision Records (ADRs) for tracking technical decisions that have a significant impact upon our codebase. See here for details:https://github.com/MetaMask/decisions

All contributors are encouraged to participate in these discussions, and all teams are encouraged to use that repository to track their own architectural decisions.

#### Threat Modeling

We use the [4 question framework](https://github.com/adamshostack/4QuestionFrame) for threat modeling substantive product changes.

### Development

#### Git workflow

The git workflow we primarily use is [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow#following-github-flow). Every repository has a single main development branch (usually called `main`), and this branch gets updated solely via pull request. The main development branch is expected to be stable and ready to release at all times.

The only place where we deviate from GitHub flow is on the client repositories, and only for releases. The MetaMask extension and mobile clients require longer manual QA cycles, during which we need a code freeze without disrupting day-to-day development, so GitHub flow was insufficient on its own. Instead we perform releases testing and stabalization on "release candidate" branches, and syncronize them with the main development branch post-release. This branching model is known as [Gitflow](https://nvie.com/posts/a-successful-git-branching-model/).

However, most contributors don't need to understand Gitflow. Just use GitHub flow and make your changes on the main branch. If your change is needed for a release, cherry-pick it onto the release candidate after the change has been merged into the main branch. All other release steps are handled by the client delivery teams.

### Automated testing

We aim to test all functionality with unit/integration tests. Additionally, we use end-to-end tests to test the "main flows" and high risk areas for each application.

### Quality Assurance

### Release and Monitoring

### Releases

We release MetaMask extension and mobile every 2 weeks. With each release, we include all changes on the main development branch.

Typically, contributors need not be concerned about which release will include their changes. Instead, focus on getting your changes accepted into the main development branch and keeping that branch stable and ready to release at all times. The extension and mobile delivery teams will ensure it gets included in the next regularly scheduled release.


### Engineering Principles

See here:

### Git Workflow

All of our repositories have a single main development branch. This branch should be "stable" at all times, ready to be released at a moment's notice.

Contributors create "feature branches" for their work, and then merge their changes back into the main branch by submitting a "pull request". There are no commits directly to main branch; every change is made via a pull request, because every change is reviewed by at least one human.

See here for more guidelines about creating and reviewing pull requests:

### Feature flags

MetaMask uses "feature flags" for incrementally developing and rolling out features. We have a few different types of feature flags:

* Development feature flags
  Used for features still under active development. Development feature flags let us implement new features one piece at a time without breaking the main development branch, and without resorting to the use of gigantic PRs or long-running feature branches (TODO: link relevant parts of PR guidelines).
* Rollout feature flags
  Used to control how a feature is rolled out to users. A rollout feature flag can be an option under the "Experimental" settings page, or it can be controlled remotely using Launch Darkly. Rollout feature flags allow users to try features out ahead of time and give early feedback, and they let us gradually release features to minimize risk or server load. They can sometimes also let us rollback a feature after we've dicsovered a bug.



### Coding guidelines

TODO: Link each coding guideline here

### Product metrics

MetaMask extension and mobile use Segment for collecting product metrics.

TODO: Mention event schema repo and process here

### Error monitoring

MetaMask extension and mobile use Sentry for tracking errors in production. Our goal is to get as close to zero error events as possible.

TODO: Link Sentry triage process

### Performance monitoring

MetaMask extension and mobile use Sentry for monitoring performance. We embed custom instrumentation in each client to track exactly the data that is most relevant to us. On the Sentry dashboard, we can see dashboards of this data (broken down by user segments as well), and we can create alerts to ensure we notice when we have a significant regression in an area we are tracking.

### Testing

TODO: General testing guidance: manual, unit, integration, e2e. Link docs for each.

### Release process

#### Issue triage

Issue triage is where we review a new issue to ensure that it is well-understood by the team and documented properly. Most critically, this is where we should recognize and escalate urgent problems. Issues should always be labelled correctly after triage as well.

Issues from external sources typically go through one initial round of triage where a team is assigned, then are triaged again by the assigned team. It's critical when assigning a team that you notify the team on Slack, and that you communicate urgency if the issue might be urgent.


TODO: Describe high-level release process, link further details

### Incident management

TODO: Describe high-level incident management process, link further details


