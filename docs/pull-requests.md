# Guide to Pull Requests

## On creating pull requests

### Use the description to explain the need and solution

When you create a pull request, fill out its description.

A well-written description:

- seeks to explain not only what was changed, but _why_
- is written for people not only within your team, but also outside it
- is written for people not only in the present, but also in the future

As you are filling out the description, use these questions as a guide:

- **What is the context behind your changes?** Remember that your reviewers might not know everything you know. Set the stage: what sort of domain-specific information can you share with them so that they can evaluate your changes more easily? (Linking to a ticket is better than nothing, but prefer to summarize the context as best you can in order to save your reviewers valuable time.)
- **What is the purpose of your changes?** What is insufficient about the way things work now? What's the user story?
- **What is your solution?** How do your changes satisfy the need? Are there any changes in particular whose purpose might not be obvious or whose implementation might be difficult to decipher? How do they work?

In the majority of cases, these questions can be answered in as little as one or two paragraphs.

Additionally, most repositories within MetaMask have a pull request template which provides a placeholder for this information (either "Description" or "Explanation").

Here are some examples of pull request descriptions that satisfy the criteria above:

- <https://github.com/MetaMask/metamask-extension/pull/19970>
- <https://github.com/MetaMask/metamask-extension/pull/18629>
- <https://github.com/MetaMask/metamask-mobile/pull/6677>
- <https://github.com/MetaMask/snaps/pull/1708>

> [!NOTE]\
> The same guidelines above apply just as well to Git commit messages as they do to pull request descriptions. In fact, if you create a pull request from a branch with only one commit, GitHub will copy your commit message into the pull request description. This means that if you focus on writing a good commit message _before_ you push up your branch, when you go to create your pull request, you're done — you don't have to spend any extra time filling out the pull request description. As a bonus, your commit will thereafter be visible in Git rather than GitHub, saving future code spelunkers time.
>
> Here are some great resources for writing good commit messages:
>
> - ["Explain the context"](https://github.blog/2022-06-30-write-better-commits-build-better-projects/#explain-the-context) from "Write Better Commits, Build Better Projects" by GitHub
> - ["My favourite Git commit"](https://dhwthompson.com/2019/my-favourite-git-commit) by David Thompson
> - ["How to Write a Git Commit Message"](https://commit.style)

### Add pull request comments

If there are specific changes within your pull request that you'd like to call your reviewers' attention to, yet you'd rather keep the pull request description succinct, open a review on GitHub and post comments on your own pull request at key locations.

#### Read more

- ["Leaving Comments on My Own Pull Requests"](https://hector.dev/2021/02/24/leaving-comments-on-my-own-pull-requests/) by Hector Castro

### Create smaller pull requests

Large pull requests are extremely painful to review. They also need to be kept up to date more frequently since they have a higher chance of creating conflicts that need to be resolved.

Ideally, a ticket should be broken down into small pieces ahead of time so to prevent so many changes from appearing in a single pull request. If, despite this effort, a pull request grows in size, then it should be broken down into logical stages.

## On reviewing pull requests

### Be compassionate

Assume good intent on the part of the author, and be mindful of constraints, tradeoffs, or priorities that may have invisibly guided the direction of the pull request in your response.

### Go beyond the surface level

It is important that changes to a codebase follows the standards and best practices of that codebase, but it is more important that it solves the underlying user story in a sound manner.

### Be curious, not curt

Rather than forcing your ideas onto the author ("delete this comment", "rename this variable"), open a dialogue instead:

- "I'm worried that this approach would ..."
- "I'm wondering if it makes sense to ..."
- "Should we ...?"
- "What do you think about ...?
- "Is it worth it to ...?"
- "Did you mean to ...?"

### Show, don't tell

Use GitHub's suggestion feature so that the author can understand your ideas and incorporate them more quickly without extending the discussion further:

    ``` suggestion
    ... add your suggestion here ...
    ```

### Highlight non-blocking comments

As you are leaving feedback, you might propose suggestions such as stylistic changes that you know rank lower in importance than others and are therefore not necessary in order to merge the pull request. You can communicate this intent to the author by preceding your comment with a prefix such as `Nitpick: ` or `Nit: `.

```
Nit: Jest has a `jest.mocked` function you could use here instead of `jest.MockFunction`. That should let you clean this up a bit if you wanted.
```

### Let go of your code

No two people truly think alike. Even when given the same exact information, they may take different approaches to solving the same problem.

For this reason, it can sometimes be challenging to evaluate someone else's pull request objectively, especially if you wrote the code that is being changed and the author has extended your code in a way that you did not intend.

In this situation, do your best to state your position, providing the author with missing context, and accept that the author has the right to make the final decision.

### Praise good work

If the author of the pull request did a great job, be sure to let them know!

### Take criticism offline

If you fundamentally disagree with the approach that the author has taken, it may be best to have a conversation with them outside of the pull request itself. You might find you can resolve differences more easily and more quickly — while preventing a public heated discussion from making you, the author, or anyone else look bad — by using an alternate form of communication, such as a video call.

### "Request changes" sparingly

When you write a review on GitHub you can assign it to one of three states: "Comment", "Request changes", or "Approved".

The "request changes" state places an X next to your review and prevents the author from merging their changes until either you approve the pull request or someone else dismisses your review. As a result, consider using it in circumstances where you truly feel that the changes being proposed cannot be merged, making sure to explain your reasoning in the review.

## On receiving feedback

### Be open to other perspectives

Understand that some people's approaches may be motivated by a different set of values than you. Their perspective may reveal something you're not seeing.

### Assume positive intent

If you receive a comment that is brusque, has a negative tone, is or difficult to decipher, do your best to handle it gracefully and ask the reviewer for more information if necessary.

### Point to updates

As you are fulfilling others' requests, respond by linking to the commit that contains the changes you made. This gives reviewers the ability to check your work and allows the discussion to reach a resolution. (Tip: You can find commits you've pushed in the "Conversation" view of the pull request. If you copy a commit ID, then GitHub will link to it when you paste it in a comment. You can also just type the commit ID in your comment and GitHub will automatically link it.)

```
Good catch! Updated in abc1234.
```

### Employ alternate forms of communication

If you sense that a conversation between you and the author is proceeding in an undesirable direction, consider reaching out to the person directly and offering to talk through your concerns on a video call.

## On maintaining pull requests

### Communicate takeovers

If you need to take over and merge someone else's branch, let them know so they aren't surprised.

### Rebase with caution

Once you've created a pull request, avoid amending, rebasing, or otherwise altering the history of the branch, but push up each new change as a new commit instead. This has a few advantages:

1. It aids reviewers in following your pull request over time. Keep in mind that they aren't as focused on your pull request as you are; they might look over your code, write some comments, leave, and then come back at a later time. The Conversation view of a pull request is an essential tool for reviewers to understand what happened while they were away. Usually, the timeline view preserves events in the same order in which they occurred. But rebasing the branch changes that order with regard to commits: specifically, if they were commingled with conversations before, now they are listed at the very end of the timeline. This can be extremely jarring.
2. It ensures that conversations on a pull request stay as they are. Rebasing is not only confusing to reviewers because it jumbles the history of the pull request, but it can also be confusing to GitHub. If a commit connected to a conversation changes enough, GitHub may think that the conversation is outdated and mark it as such. This is misleading and may cause the author of the pull request to start ignoring the conversation.
3. It creates a smooth workflow with collaborators. If you're working with someone else but you change the history of the branch, they may have to blow away their version of the branch and re-fetch it. This creates friction, causes frustration, and wastes time.

Make sure that when you click the button on GitHub to update a branch with its base branch, you do not choose the "rebase" option.

If you absolutely have to change the history of a pull request's branch, inform your reviewers and/or collaborators appropriately.

## On merging pull requests

### Clean up the commit message

When you press the big green "Squash & Merge" button on GitHub, you will be presented with an option to modify the commit message before you merge.

The default "Squash & Merge" message can vary between repositories ([it is configurable in the repository settings](https://github.blog/changelog/2022-08-23-new-options-for-controlling-the-default-commit-message-when-merging-a-pull-request/)). Depending on the circumstance, it might contain the pull request description, the commit message, or a concatenated list of commit messages.

Take the time to review this message to ensure that it describes the change well. For example, if the default message is a list of commits, you may want to replace it with the pull request description.

However, please **do not modify the format of the commit title**. Some of our automated scripts depend on the title being formatted exactly as "Pull request title (#number)"; deviating from that could cause problems later on when preparing the next release. If you find that the title doesn't describe the change well, you can edit the title before merging.
