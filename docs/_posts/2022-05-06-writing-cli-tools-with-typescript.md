# Writing CLI tools with TypeScript

## Introduction

There are quite a couple of technology options when it comes to writing a *command line interface* (CLI) tool.

In fact, most of the modern languages at the very least provide some kind of out of the box support for command line argument parsing. 

Typically modern language ecosystems also have at least one mature and feature rich CLI library.

A few of the languages and libraries that I've used to write CLIs are listed below

* [clap](https://docs.rs/clap/latest/clap/) (Rust)
* [Click](https://docs.python-guide.org/scenarios/cli/) (Python)
* [Picocli](https://picocli.info/picocli-2.0-groovy-scripts-on-steroids.html) (Groovy)
* [Commander.js](https://github.com/tj/commander.js) (Node.js)



So, why would we prefer writing a CLI in TypeScript given all these other choices?

Well, TypeScript is an overall good choice for a couple of reasons: 

1. **compatibility** with JavaScript (TypeScript is a superset of—and by that token compatible with—JavaScript),
2. **popularity** (TypeScript was ranked 7th in the [2021 Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey/2021#section-most-popular-technologies-programming-scripting-and-markup-languages) with JavaScript being 1st),
3. **tooling choice** (the Node.js ecosystem provides [quite a lot of CLI libraries](https://openbase.com/categories/js/best-javascript-cli-libraries)),
4. **developer friendliness** (TypeScript is a scripted language but also has an advanced type system and static type checking which helps writing high quality code),
5. **performance** (since TypeScript code is in the end typically run by Node.js it is in general sufficiently performant thanks to the [v8](https://v8.dev/) engine).



Nonetheless, a typically non-trivial part in any TypeScript project is set up. Using e.g. Python with the [Click](https://click.palletsprojects.com/en/8.1.x/) library, requires little more than writing a file with a [main function](https://docs.python.org/3/library/__main__.html#packaging-considerations). This set up complexity is of course due to the nature of TypeScript as a superset of JavaScript. To run TypeScript code, you typically transpile to JavaScript and then execute the JavaScript code with Node.js. (Yes, even [`ts-node`](https://github.com/TypeStrong/ts-node) does that.) So, before running a program written in TypeScript, we we need to transpile to JavaScript first. This requires tooling and therefore setup.



Preferably, we also want to have a single, packaged binary as the result of building our project rather than a script file to reduce installation overhead on part of the user. 

To assure high code quality we also typically want to include code formatting, linting, and tests.

## Basic development flow

To hit the ground running, we use a starter project which already contains all the set up required for running our code, formatting, linting, testing and packaging to a binary and install its dependencies. 

> :information_source: The starter project for can be found on my [GitHub](https://github.com/fkurz/typescript-cliutil-starter) and on [npm](https://www.npmjs.com/package/@fkurz/typescript-cliutil-starter/). 

```bash
git clone git@github.com:fkurz/typescript-cliutil-starter.git my-cli
cd my-cli 
npm i
```

Our starter project typescript-cliutil-starter uses 

* the [*commander*](https://www.npmjs.com/package/commander) package for argument parsing and executing commands, 

* code linting and formatting with [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint) and [prettier](https://prettier.io/), 
* [Jest](https://jestjs.io/) as test runner as well as testing and assertion library, and 
* [pkg](https://github.com/vercel/pkg) to build a stand alone binary.



To illustrate the development flow, we now add an example subcommand and test it. 

For example, we could add a command _yell_ which is exactly like the *say* command (that already exists in the project) but a bit less subtle.

To do this, we add a file _src/yell.ts_ and then add the following code.

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

We also have to register our yell command with the main CLI command in src/cmd.ts, i.e. add an import statement for our subcommand and register it using *Commander.addCommand* 

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
```

Having done that, we can run a quick test on our command using the `dev` script in package.json. The dev script simply executes our TypeScript code with    [`ts-node`](https://github.com/TypeStrong/ts-node).

```
$ npm run dev -- yell 'hey!'

> @fkurz/typescript-cliutil-starter@1.0.0 dev /private/tmp/my-cli
> ts-node src/main.ts "yell" "hey!"

HEY!
```

We can also add a quick test by adding _src/yell.test.ts_ wit the following content

```typescript
import buildCmd from "./cmd";

beforeEach(() => {
  jest.spyOn(process, "exit").mockImplementation();
  jest.spyOn(console, "log").mockImplementation();
});

const buildArgs = (...args: string[]) => ["node", "cmd", ...args];

describe("Yell subcommand", () => {
  it("Should say 'HEY!' when passed 'hey!'", () => {
    const cmd = buildCmd();
    cmd.parse(buildArgs("yell", "hey!"));

    expect(console.log).toHaveBeenCalledWith("HEY!");
  });
});
```

Since our Jest configuration expects JavaScript test files, we first have to transpile our code with the build script using the TypeScript compiler *tsc*. The transpiled JavaScript source code is put into the dist/ folder.

```bash
npm run build

> @fkurz/typescript-cliutil-starter@1.0.0 build /private/tmp/my-cli
> tsc --build
```

We can now run our test suite with Jest using the `test` script. New test files will automatically get recognized by Jest if they have—as in our case—the suffix _.test.js_

```
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

As a final step, we can build a native binary by executing `npm run package`. The `pkg` tool will then bundle all source code an dependencies into a single binary file *bin/main*. 

```bash
$ npm run package

> @fkurz/typescript-cliutil-starter@1.0.0 package /private/tmp/my-cli
> pkg dist/main.js --no-bytecode --public-packages '*' --public --target host --output bin/main

> pkg@5.3.1
```

Before running the program, we have to make it executable 

```bash
chmod +x ./bin/main
```

Now we can simply run the binary from the shell without a Node.js installation and without any dependency installations whatsoever.

```bash
$ ./bin/main yell 'hey!'
HEY!
```

## Summary

TypeScript is a good choice for writing a CLI utility program due to various factors including being a developer friendly language that also has static checks to assure code quality and moreover because it offers a wide range of tooling choices for command line programs within its ecosystem.



As shown above, developing a CLI uility developed in TypeScript does not preclude the option of exporting a stand alone, native binary if we use a tool like pkg to bundle our program and dependencies into an application binary.

## About the Author: Friedrich Kurz

I have been working for MaibornWolff as a full-time software engineer for 2.5 years now. In my current project, I'm working on AWS cloud infrastructure development for a client's web platform. I would describe myself as a technology generalist with a broad range of interests rather than a technology specialist but I'm also very interested in functional programming and general methods of writing correct, clean, and maintainable code.



Do you have any questions, suggestions, or comments? Get in touch with me [via mail](https://aemail.com/Z4YQ)!

