# Guidelines for Writing Guidelines

## Use this document as a guide

This document is not only a set of guidelines, but also a template for other guidelines documents.

## Be direct

Guidelines should be as easy to understand and follow as possible. Think of each guideline like a commit: start with a concise summary, preferably a direct command, then use a paragraph or two to elaborate.

## Be succinct

Remove extraneous language in both the guideline itself and the elaboration.

## Bolster your guideline if necessary

If a discussion exists in a public GitHub repository which provides context around a guideline, link to that discussion. Or, if a book, blog post, or related work exists which helps to explain the purpose of a guideline, link to the resource.

Use a "Read more" section to collect multiple resources:

``` markdown
### Read more

- [Link 1](https://example1.com)
- [Link 2](https://example2.com)
- [Link 3](https://example3.com)
```

## Provide examples

It is far easier to understand the purpose and effects of a guideline not by reading about it, but by observing an example.

A common way to provide an example is to offer a dichotomy, a passage of code which falls outside the guideline followed by a passage of code that conforms to it. Understanding that some decisions are arbitrary, take care not to use language like "bad"/"good", "incorrect"/"correct", etc., which may sound judgmental. Rather than assigning a grade to the code in question, use emoji:

    🚫

    ``` javascript
    const x = 42;
    ```

    ✅

    ``` javascript
    const THE_MEANING_OF_LIFE = 42;
    ```

## Mark guidelines with options accordingly

By default, a guideline is intended to be followed unless there is a good reason not to do so.

Occasionally, however, a guideline may offer a set of options among which the reader may choose. To designate this type of guideline, prefix the header with 💡.
