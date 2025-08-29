# Release process
This diagram below represents the release process for MetaMask Extension and MetaMask Mobile clients.

Note:
- The `stable` branch is currently called `master` on Extension repo, but it will be renamed soon.
- The `release/x.y.z` branch is currently called `Version-vx.y.z` on Extension repo, but it will be renamed soon.

```mermaid
graph TD

%% Nodes outside subgraphs %%
RUN[Runway] -->|every 2 weeks| CURRENT1
RE[Release Engineer] -->|manual trigger| GA[Release Engineer triggers 'Create Release Pull Request' GitHub Action]
GA -->|create PR| CURRENT2
GA -->|create PR| changelog1[GitHub Action creates x.y.z changelog PR]
changelog1 -->|update PR| changelog2[Release Engineer reviews and adjusts x.y.z changelog PR]
changelog2 -->|merge PR| CURRENT5
GA --> |create PR| bump1[GitHub Action creates version bump PR]
bump1 -->|merge PR| MAIN1
CURRENT6 --> BUG1[A bug is found]
BUG1 --> BUG2[A fix is done on 'main' branch]
BUG2 --> CURRENT7
ENG[Engineer] -->|create PR| FEAT[Implement new features]
FEAT -->|merge PR| MAIN1

%% Subgraphs %%
subgraph Main [Main developement branch: 'main']
    style Main fill:#4d0808,stroke:#000,stroke-width:2px,color:#fff
    MAIN1[All changes are first made on 'main' branch]
end

subgraph Previous [Previous release branch: 'release/x.y-1.z']
    style Previous fill:#08084d,stroke:#000,stroke-width:2px,color:#fff
    PREVIOUS1[Previous release is merged into 'stable' branch]
    PREVIOUS1 -->|create PR| PREVIOUS2[Release Engineer creates stable sync PRs]
    PREVIOUS2 -->|merge PR| MAIN1
end

subgraph HotFix [Hotfix release branch: 'release/x.y-1.z+1']
    style HotFix fill:#08084d,stroke:#000,stroke-width:2px,color:#fff
    HOTFIX1[Hot fix release is merged into 'stable' branch]
    HOTFIX1 -->|create PR| HOTFIX2[Release Engineer creates stable sync PRs]
    HOTFIX2 -->|merge PR| MAIN1
end

subgraph Current [Current release branch: 'release/x.y.z']
    style Current fill:#08084d,stroke:#000,stroke-width:2px,color:#fff
    MAIN1 -->|every 2 weeks| CURRENT1[Runway automatically creates a new release branch based off of 'main' branch, called 'release/x.y.z']
    CURRENT1 --> CURRENT2[GitHub Action creates x.y.z release PR]
    CURRENT2 -->|every time a commit is added to 'release/x.y.z' branch| CURRENT3[Release build is automatically created and posted on the x.y.z release PR]
    CURRENT3 --> CURRENT4[Release Engineer creates and merges stable sync PR into 'release/x.y.z branch']
    PREVIOUS2 -->|every time a release is merged into 'stable'| CURRENT4
    HOTFIX2 -->|every time a release is merged into 'stable'| CURRENT4
    CURRENT4 --> CURRENT5[Changelog is added]
    CURRENT5 --> CURRENT6[Release is tested by all teams]
    CURRENT7[Release Engineer cherry-picks fixes on 'release/x.y.z' branch]
    CURRENT6 --> CURRENT8[Release is approved by all teams]
    CURRENT7 --> CURRENT8
    CURRENT8 --> CURRENT9[Release is approved by Release Engineer, Release QA, and Release Manager]
end

subgraph Stable [Stable branch: 'stable']
    style Stable fill:#26084d,stroke:#000,stroke-width:2px,color:#fff
    CURRENT9 -->|merge PR| STABLE1[x.y.z release PR is merged into 'stable' branch]
    STABLE1 -->|every time a commit is added to 'stable' branch| STABLE2[x.y.z production build is automatically created]
    STABLE2 --> STABLE3[Release Engineer submits x.y.z production build to the store]
end

subgraph Next [Next release branch: 'release/x.y+1.z']
    style Next fill:#08084d,stroke:#000,stroke-width:2px,color:#fff
    STABLE3 --> NEXT1[Runway automatically creates a new release branch from main, called 'release/x.y+1.z']
end