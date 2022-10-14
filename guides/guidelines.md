# Guidelines for Writing Guidelines

## Use this template as a guide

This document is not only a set of guidelines, but also a template for other guidelines documents.

## Be direct

Guidelines should be as easy to understand and follow as possible. Think of each guideline like a commit: start with a concise summary, preferably a direct command, then use a paragraph or two to elaborate.

## Be succinct

Remove extraneous language in both the guideline itself and the elaboration (see [*The Elements of Style*](https://www.gutenberg.org/files/37134/37134-h/37134-h.htm#Rule_13) for help).

## Bolstering your guideline, if necessary

If a discussion exists in Slack or GitHub which spawned your guideline, link to that discussion. Or, if a book, blog post, or related work exists which can help to shine light on your guideline's utility, link to the resource.

Although not strictly necessary for arbitrary guidelines — where the problems they are solving have multiple possible solutions and none of the solutions have a significant advantage over the other — or for axiomatic guidelines — where the values they express are assumed to be good, such that the absence of these values is assumed to be bad — including a context or rationale with your guideline is still helpful for onboarding.

Here is an example:

``` markdown
## Use snake-case to name files and directories.

[**View discussion on GitHub**](https://example.com).
```

## Provide examples

It is far easier to understand a principle not by reading about it, but by observing its effects.

A common way to provide an example is to offer a dichotomy: a passage of code which does not follow the guideline followed by a passage of code that does. Understanding that some decisions are arbitary, care must be taken not to use language like "bad" or "good" which could be taken as a slight against the reader's skill level. Rather than assigning a grade to the code in question, tell the reader what to do.

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

## You can change these!

The guidelines in this repository are not set in stone. If you disagree with a guideline, or you notice a guideline that's missing, feel free to create a pull request.
