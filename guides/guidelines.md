# Guidelines for Writing Guidelines

## 🙏 Use this document as a guide

This document is not only a set of guidelines, but also a template for other guidelines documents.

## 🙏 Communicate the importance of your guideline

Guidelines may differ in how closely they are intended to be followed. To designate the importance of a guideline, include an emoji in its header to assign it to one of three categories:

1. **🙏 (Standard):** These are guidelines that it really behooves everyone to follow. There shouldn't be a reason not to follow them, but anyone who feels that a guideline should be changed or removed should start a discussion in a pull request.
2. **🌳 (Recommended):** These are guidelines that everyone ought to follow, though teams or projects may opt to bypass or change them if they have a good reason to do so.
3. **💡 (Suggested):** These are guidelines that offer one way to solve a problem among multiple, equally valid alternatives. When you are writing such a guideline, enumerate these alternatives and suggest one of them for the reader to follow.

## 🙏 Be direct

Guidelines should be as easy to understand and follow as possible. Think of each guideline like a commit: start with a concise summary, preferably a direct command, then use a paragraph or two to elaborate.

## 🙏 Be succinct

Remove extraneous language in both the guideline itself and the elaboration (see [*The Elements of Style*](https://www.gutenberg.org/files/37134/37134-h/37134-h.htm#Rule_13) for help).

## 🙏 Bolster your guideline if necessary

If a discussion exists in Slack or GitHub which spawned your guideline, link to that discussion. Or, if a book, blog post, or related work exists which can help to shine light on your guideline's utility, link to the resource.

Although not strictly necessary for arbitrary guidelines (which aim to solve problems with multiple, equal-advantage solutions) or for axiomatic guidelines (which abide by values assumed to be good over those assumed to be bad), including a context or rationale with your guideline is still helpful for onboarding.

Here is an example:

``` markdown
## Use snake-case to name files and directories.

[**View discussion on GitHub**](https://example.com).
```

## 🙏 Provide examples

It is far easier to understand a principle not by reading about it, but by observing its effects.

A common way to provide an example is to offer a dichotomy: a passage of code which does not follow the guideline followed by a passage of code that does. Understanding that some decisions are arbitary, take care not to use language like "bad" or "good" which could be misread by others as a slight against them. Rather than assigning a grade to the code in question, tell the reader what to do.

For instance, instead of this:

~~~ markdown
**Bad:**

``` javascript
const x = 42;
```

**Good:**

``` javascript
const THE_MEANING_OF_LIFE = 42;
```
~~~

say something like this:

~~~ markdown
Instead of this:

``` javascript
const x = 42;
```

say this:

``` javascript
const THE_MEANING_OF_LIFE = 42;
```
~~~

## 🙏 Be empowered to propose your ideas/suggestions

The guidelines in this repository are not set in stone, and none of us is the gatekeeper. If you disagree with a guideline, or you notice a guideline that's missing, feel free to open a discussion in a pull request.
