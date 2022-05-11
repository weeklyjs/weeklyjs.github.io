---
layout: post
title:  Writing CLI tools with TypeScript
date:   2022-05-06 08:00:13 +0100
categories: typescript
author: Friedrich Kurz
---
There are quite a couple of technology options when it comes to writing a *command-line interface* (CLI) tool. Most modern languages at the very least provide some kind of out-of-the-box support for command-line argument parsing. In addition, the ecosystems of modern languages also typically contain at least one mature and feature-rich CLI library. 

TypeScript is a good choice for writing CLI tools for a couple of reasons including developer-friendliness, static checks, as well as a broad choice of tooling options. As with every TypeScript project, there is however an initial setup hurdle. We'll have a look at how to set up a TypeScript CLI project with linting, formatting, testing, and packaging to a standalone binary in this post.

To give a few examples, here are some of the libraries and languages I've used recently to write CLIs

- [clap](https://docs.rs/clap/latest/clap/) (Rust)
- [Click](https://docs.python-guide.org/scenarios/cli/) (Python)
- [Picocli](https://picocli.info/picocli-2.0-groovy-scripts-on-steroids.html) (Groovy)
- [Commander.js](https://github.com/tj/commander.js) (JavaScript)

So, why would we prefer writing a CLI in TypeScript given all these other choices?

Well, TypeScript is an overall good choice for a couple of reasons: 

1. **compatibility** with JavaScript (TypeScript is a superset of—and by that token compatible with—JavaScript),
2. **popularity** (TypeScript was ranked 7th in the [2021 Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey/2021#section-most-popular-technologies-programming-scripting-and-markup-languages) with JavaScript being 1st),
3. **tooling choice** (the Node.js ecosystem provides [quite a lot of CLI libraries](https://openbase.com/categories/js/best-javascript-cli-libraries)),
4. **developer friendliness** (TypeScript is a scripted language but also has an advanced type system and static type checking which helps writing high-quality code),
5. **performance** (since TypeScript code is, in the end, typically run by Node.js, it is in general sufficiently performant thanks to the [v8  engine](https://v8.dev/)).

Nonetheless, a typically non-trivial part of any TypeScript project is the setup. Writing a CLI with e.g. Python and the [Click](https://click.palletsprojects.com/en/8.1.x/) library requires little more than writing a file with the [main function](https://docs.python.org/3/library/__main__.html#packaging-considerations). This higher setup complexity is of course owed to the fact that TypeScript is a superset of JavaScript. To run TypeScript code, you typically transpile it to JavaScript and then execute the JavaScript code with Node.js. (Yes, even [ts-node](https://github.com/TypeStrong/ts-node) does that.) So, before running a program written in TypeScript, we need to transpile it to JavaScript first. This requires tooling and therefore setup.

Preferably, we also want to have a single, packaged binary as the result of building our project, rather than a script file, to reduce installation overhead on part of the user. To assure a high level of code quality, we also typically want to include code formatting, linting, and tests.

## Getting started writing a TypeScript CLI

To hit the ground running, I created a starter project which already contains all required boilerplate and dependencies for running, formatting, linting, testing, and packaging the code to a standalone binary. 

> ℹ️ The starter project can be found on my [GitHub](https://github.com/fkurz/typescript-cliutil-starter) and via [npm](https://www.npmjs.com/package/@fkurz/typescript-cliutil-starter/). 

We first clone it and install dependencies.

```bash
git clone git@github.com:fkurz/typescript-cliutil-starter.git my-cli
cd my-cli 
npm i
```

Our starter project uses 

- the [*commander*](https://www.npmjs.com/package/commander) package for argument parsing and executing commands, 
- code linting and formatting with [eslint](https://eslint.org/) and [prettier](https://prettier.io/), 
- [Jest](https://jestjs.io/) as test runner and assertion library, and 
- [pkg](https://github.com/vercel/pkg) to build a stand-alone binary.

## Basic development flow

To illustrate the development flow, we now add an example subcommand `yell` and test it. `yell` is similar to the `say` command (that already exists in the project) but a bit less subtle. 

To do this, we add a file `src/yell.ts` and then add the following code.

```typescript
// src/yell.ts
import { Command } from "commander";

export default (): Command =>
 new Command()
  .name("yell")
  .description("Say the word passed as the first argument but louder")
  .argument("<word>")
  .action((word: string) => console.log(word.toLocaleUpperCase()));
```

We also have to register our yell command with the main CLI command in `src/cmd.ts`, i.e. add an import statement for our subcommand and register it using *Commander.addCommand* 

```typescript
// src/cmd.ts
// … 
import buildYellCmd from "./yell";
// …
export default (): Command => {
  const command = new Command()
    .option("-g, --greet", `Say ${HELLO}`, false)
    .addCommand(buildSayCmd())
    .addCommand(buildYellCmd())
    .addHelpCommand()
    .showHelpAfterError();

  // …

  return command;
};
```

Having done that, we can do a quick test run using the `dev` script defined in `package.json`. The `dev` script simply executes our TypeScript code with [ts-node](https://github.com/TypeStrong/ts-node).

```bash
$ npm run dev -- yell 'hey!'

> @fkurz/typescript-cliutil-starter@1.0.0 dev /private/tmp/my-cli
> ts-node src/main.ts "yell" "hey!"

HEY!
```

We can also add a test by adding `src/yell.test.ts` with the following content

```typescript
import buildCmd from "./cmd";

beforeEach(() => {
  jest.spyOn(process, "exit");
  jest.spyOn(console, "log");
});

describe("Yell subcommand", () => {
  it("Should say 'HEY!' when passed 'hey!'", () => {
    const cmd = buildCmd();
    cmd.parse(["node", "cmd", "yell", "hey!"]);

    expect(console.log).toHaveBeenCalledWith("HEY!");
  });
});
```

Since our Jest configuration expects JavaScript test files, we first have to transpile our code with the build script using the TypeScript compiler `tsc`. The transpiled JavaScript source code is put into the `dist/` folder.

```bash
$ npm run build
> @fkurz/typescript-cliutil-starter@1.0.0 build /private/tmp/my-cli
> tsc --build
```

We can now run our test suite with Jest using the test script. New test files will automatically get recognized by Jest if they have—as in our case—the suffix `*.test.js*`.

```bash
$ npm run test
> @fkurz/typescript-cliutil-starter@1.0.0 test /private/tmp/my-cli
> jest

...

Test Suites: 3 passed, 3 total
Tests:       8 passed, 8 total
Snapshots:   0 total
Time:        0.41 s, estimated 1 s
Ran all test suites.
```
As a final step, we can build a native binary by executing npm run package. This script will conveniently bundle all source code and dependencies into a single, stand-alone binary file `bin/main` using the `pkg` tool. 

```bash
$ npm run package

> @fkurz/typescript-cliutil-starter@1.0.0 package /private/tmp/my-cli
> pkg dist/main.js --no-bytecode --public-packages '*' --public --target host --output bin/main

> pkg@5.3.1
```

The packaged binary is automatically made executable using the `postpackage` script.
Packaging our CLI into a standalone binary has the major advantage that we can now simply run our CLI as a binary from the shell without a Node.js installation and without having to download any dependency whatsoever.

```bash
$ ./bin/main yell 'hey!'
HEY!
```

## Summary

TypeScript is a good choice for writing a CLI tool due to various factors.

Not only is TypeScript a developer-friendly language but it also has static checks to assure code quality and its ecosystem offers a wide range of tooling choices for command-line programs.

Additionally, as shown above, developing a CLI utility written in TypeScript does not preclude the option of exporting a stand-alone, native binary if we use a tool like `pkg` to bundle our program and dependencies into an application binary.

Do you have any questions, suggestions, or comments? Get in touch with the author [via mail](https://aemail.com/Z4YQ)!

## About the Author: Friedrich Kurz

Friedrich has been working for MaibornWolff as a full-time software engineer for 2.5 years. In his current project, he's helping to build the AWS cloud infrastructure for a client's web platform. Friedrich considers himself a technology generalist with a broad range of interests rather than a technology specialist. He's, however, also very interested in functional programming and general methods of writing correct, clean, and maintainable code.
