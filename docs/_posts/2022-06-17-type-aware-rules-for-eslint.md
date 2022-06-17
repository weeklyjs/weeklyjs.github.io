---
layout: post
title: "Type-Aware Rules for ESLint"
date: 2022-06-17 10:00:00 +0100
categories: typescript
author: Raphael Pigulla
---

# Type-aware rules for ESLint

In a previous article my colleague [Tobias Walle](https://www.maibornwolff.de/team/tobias-walle) already discussed how [Static code analysis in Node.js](https://weeklyjs.io/javascript/2022/04/22/static-code-analysis.html) can be used to improve the quality of your code, specifically with [ESLint](https://eslint.org/). Most TypeScript developers are probably already aware of the [eslint-plugin](https://typescript-eslint.io/) which provides a plethora of additional rules specifically for TypeScript.

What is slightly less known is that you can opt in to [type-aware rules](https://typescript-eslint.io/docs/linting/#type-aware-rules), albeit at some [cost to performance](https://typescript-eslint.io/docs/linting/type-linting#how-is-performance). It is highly recommended to enable this feature unless the size of your code base makes the use of it prohibitively expensive - in which case you might want to defer these checks to a pre-commit hook.

Setting this up is slightly more tricky than your average ESLint-plugin, but the [documentation](https://typescript-eslint.io/docs/linting/type-linting) does a great job of guiding you through it. Just make sure you read the [troubleshooting](https://typescript-eslint.io/docs/linting/type-linting#troubleshooting) section because chances are you will run into issues if you don't. Or so I am told 🙄.

So what's the benefit of all of this? Consider the following (admittedly somewhat silly) piece of code, which will pass conventional linting just fine:
```typescript
async function boom(): Promise<number> {
    throw '💥'
}

boom()
```
Eagle-eyed developers will quickly spot at least three issues with this (besides the pointlessness of the function 😉):

1) the function is `async` even though it does not need to be
2) the call to `boom` is not `await`ed.
3) the function throws a string literal, not an an [Error object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)

This is where the type-aware rules [`require-await`](https://typescript-eslint.io/rules/require-await), [`no-floating-promises`](https://typescript-eslint.io/rules/no-floating-promises) and [`no-throw-literal`](https://typescript-eslint.io/rules/no-throw-literal), respectively, come to the rescue. Please note that the latter is not enabled by default if your configuration only extends [`plugin:@typescript-eslint/recommended-requiring-type-checking`](https://typescript-eslint.io/docs/linting/configs#recommended-requiring-type-checking), you will either have to enable it manually or also use [`plugin:@typescript-eslint/strict`](https://typescript-eslint.io/docs/linting/configs#strict).

## About the author: Raphael Pigulla

Raphael has more than ten years of experience in software development and architecture. He specializes in backend development with Node.js and TypeScript. Raphael joined [MaibornWolff](https://maibornwolff.de) in 2019.
