---
layout: post
title:  "Static code analysis in Node.js"
date:   2022-04-22 08:00:13 +0100
categories: javascript
author: Tobias Walle
---

JavaScript is a very powerful language.
This has a lot of advantages, but it can also decrease the readability of your code if misused.

Static code analysis can be used to detect common errors in your code and keep the style consistent 
across your team. In this article I want to share some of the tools that I can recommend.

## Typescript
[Typescript](https://www.typescriptlang.org/) is a statically typed superset 
of JavaScript. By attaching the type information to the variables and  
functions in your code, it helps finding errors before executing the 
code. It also boosts your productivity by improving the autocompletion of your editor.
The best thing is, most of the time Typescript can infer the type of a variable and you don't need 
to declare it. You can even [check the types of a normal JavaScript project](https://www.typescriptlang.org/docs/handbook/intro-to-js-ts.html).

## Prettier
[Prettier](https://prettier.io/) formats your code by a very opinionated rule set.
It avoids unnecessary changes caused by different formatting in your git commits.
This is especially useful in code reviews, as you don't get distracted by bloated diffs.

## Eslint
[Eslint](https://eslint.org/) is the de facto standard linter for JavaScript projects.
It comes with a large ruleset and is easily extendable with plugins.
I would recommend to start with the [recommended rules](https://eslint.org/docs/rules/) by setting 
`"extends": "eslint:recommended"` in the configuration and enable/disable individual rules as needed. 

If you want to use Eslint with Typescript, you should install and configure the [Typescript Eslint Parser](https://www.npmjs.com/package/@typescript-eslint/parser).

The following plugins might be useful to you:
- [eslint-config-prettier](https://www.npmjs.com/package/eslint-config-prettier)
  disables all the eslint rules, that might conflict with the prettier formatting.
- [eslint-plugin-import](https://www.npmjs.com/package/eslint-plugin-import)
  provides a set of rules to manage the imports in your project. I especially recommend
  the [order](https://github.com/benmosher/eslint-plugin-import/blob/HEAD/docs/rules/order.md) rule,
  which automatically sorts your imports. This also avoids unnecessary diffs in your pull requests.
- [eslint-plugin-unused-imports](https://www.npmjs.com/package/eslint-plugin-unused-imports).
  Unused imports can increase your application size and reduce readability. With this plugin you can automatically 
  remove imports with no reference in your code.
- The default eslint rules are great, but 
  [eslint-plugin-unicorn](https://www.npmjs.com/package/eslint-plugin-unicorn) 
  adds a lot more. 
  For example with the [prevent-abbreviations](https://github.com/sindresorhus/eslint-plugin-unicorn/blob/HEAD/docs/rules/prevent-abbreviations.md) rule,
  you can ban abbreviations that might reduce clarity of you code.
  I suggest that you use the recommended configuration of this plugin.
- [eslint-plugin-promise](https://www.npmjs.com/package/eslint-plugin-promise).
  Promises and async/await are great and really help to keep your async JavaScript code
  clean. This plugin detects common pitfalls and bad practices while using Promises
  and async/await.
- [eslint-plugin-node](https://www.npmjs.com/package/eslint-plugin-node) extends ESLint
  with some Node.js specific rules. For example, it can enforce the use of the new promise
  based 
  [fs](https://nodejs.org/dist/latest-v10.x/docs/api/fs.html#fs_fs_promises_api) 
  module with the 
  [node/prefer-promises/fs](https://github.com/mysticatea/eslint-plugin-node/blob/HEAD/docs/rules/prefer-promises/fs.md) 
  rules.
- [eslint-plugin-json](https://www.npmjs.com/package/eslint-plugin-json) includes JSON specific rules.
  This is especially useful, to prevent malformatted json configurations.
- Last but not least, the
  [eslint-plugin-eslint-comments](https://www.npmjs.com/package/eslint-plugin-eslint-comments)
  detects the abuse of eslint escape comments like `/* eslint-disable */`. For example
  it warns you, if the exception is to general or redundant.

Start with the recommended settings of each plugin and enable or disable individual rules based
on your project needs. An exception is the 
[eslint-plugin-import](https://www.npmjs.com/package/eslint-plugin-import).
As the individual rules are relatively slow, I would just pick the rules that 
are worth the performance impact. You can measure the performance impact of each rule
by setting the environment variable `TIMING=1` 
([source](https://stackoverflow.com/questions/38458067/which-eslint-rules-in-my-config-are-slow)). 
I would also recommend setting `--cache` flag of eslint to improve performance.

That's it! I hope I could help you to improve the static analysis 
of your JavaScript project. Do you think I missed something? 
Please let me know in the comments.
  
## About the Author: Tobias Walle

Tobias works as a full-stack developer at MaibornWolff since 2016. He works regularly with several languages, including Typescript, Kotlin and Rust
and loves to deep dive into new technical topics. 




