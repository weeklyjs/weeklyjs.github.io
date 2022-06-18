---
layout: post
title:  Types as comments for plain JavaScript
date:   2022-06-16 08:00:13 +0100
categories: typescript, javascript
author: Friedrich Kurz
---
Today we're looking into the potential future of JavaScript by reading up on the types as comments proposal that seeks to introduce static type information as part of the core JavaScript language.

## Types as comments

[Types as comments][tc39-types-as-comments] was proposed by the [TC39][tc39] technical committee for JavaScript in late 2020. It proposes the addition of syntax for **type information** as a core, but optional, part of the JavaScript language. 

Adding type information syntax to the language enables the potential use of static type checkers to prevent certain classes of bugs. That is to say, static type checking would be possible without having to develop in another statically typed language (such as *TypeScript* or *Flow*) that transpiles to JavaScript. 

For JavaScript developers, this promises at least two things: 
1. a faster development loop (i.e. without a transpilation step), and
2. free choice of tooling for type checking.

## Why static type checking?

### TypeError: Cannot read properties of undefined (reading 'bar')

The case for static type checking is not particularly hard.

As JavaScript developers, we probably all know the language's famously weak and dynamic type system which has no problem with letting funny code like the following pass to the interpreter without throwing errors (all examples were run with Node v14.19.3)

```bash
$ node -e 'console.log(3 + [])'
3
$ node -e 'console.log({}.foo)' 
undefined
```

> :information_source: Have a look at [Programming TypeScript (p.3)](https://www.google.de/books/edition/Programming_TypeScript/Y-mUDwAAQBAJ?hl=en&gbpv=1&pg=PA3&printsec=frontcover) for more examples.
> 

Obviously, even though the `+` operator may be overloaded to add strings and arrays, it doesn't make a lot of sense to do that in the first place. Neither does accessing a custom property `foo` on a, very obviously, empty object. The developers of JavaScript, in a very questionable attempt to make developer lives easier in the short run, opted to take care of these kinds of problems by

1. in the first case applying [type coercion](https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion), and 
2. in the second case by defaulting to the special value `undefined` that models undefined values.

If you have ever ripped your hairs trying to find the cause of the infamous "TypeError: Cannot read property 'bar' of undefined" runtime error, e.g.
```bash
$ node -e 'console.log({}.foo.bar)'
[eval]:1
console.log({}.foo.bar)
                   ^

TypeError: Cannot read properties of undefined (reading 'bar')
    at [eval]:1:20
    at Script.runInThisContext (node:vm:129:12)
    at Object.runInThisContext (node:vm:305:38)
    at node:internal/process/execution:76:19
    at [eval]-wrapper:6:22
    at evalScript (node:internal/process/execution:75:60)
    at node:internal/main/eval_string:27:3
```
you probably understand why, in the long run, you want to catch these kinds of problems **before** running your code. Preferably even, you want to know this immediately when you type the code into your editor of choice.

### TypeScript to the rescue

Part of the appeal of TypeScript is that it adds this additional layer of **static**—i.e. before runtime—safety to your JavaScript code.

> "The main benefit of TypeScript is that it can highlight unexpected behavior in your code, lowering the chance of bugs."
> 
> — [TypeScript in 5 Minutes](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)

Let's now, for example, directly run the above code examples with TypeScript (using `ts-node`).

```bash
$ ts-node -e 'console.log(3 + [])'
[eval].ts:1:13 - error TS2365: Operator '+' cannot be applied to types 'number' and 'never[]'.

1 console.log(3 + [])
              ~~~~~~
$ ts-node -e 'console.log({}.foo.bar)'
[eval].ts:1:16 - error TS2339: Property 'foo' does not exist on type '{}'.

1 console.log({}.foo.bar)
                 ~~~
```

As we can see, this type of error can easily be detected before running the code by applying type checking. In the first case, TypeScript tells us that we likely wanted a number on the right hand side of the `+` operator (and not add a number and an array). 

In the second case, we are informed that empty objects do not magically come with an arbitrary property called `foo`. 



> NB: Admittedly, the phrasing of the first error message is a little bit off because we **can** actually apply the `+` operator to `number` and `array` types (in our case yielding a `number` value `3` as shown above). By [specification](https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-addition-operator-plus), however, it is only overloaded for number addition and string concatenation. Moreover, it makes very little sense to "add" numbers and arrays, so the right hand side value probably was intended to be a number as well.



These two properties, i.e.

1. the plus operator is only used for number addition and string concatenation and
2. custom properties of objects are not accessed before they are set

are very reasonable constraints for good JavaScript programs. If they don't hold, we most likely have found some kind of program bug or otherwise bad code. Consequently, it's extremely helpful that TypeScript can assert (or, disconfirm) these properties for us before we run the code. 



Another terrific aspect of TypeScript's type checking is that we don't actually have to add a lot of type information. In simple cases like the above, TypeScript can simply guess the types in absence of type information via [type inference](https://www.typescriptlang.org/docs/handbook/type-inference.html). 



Unsurprisingly, a lot of JavaScript developers have come to appreciate TypeScript for these and other reasons and it is considered one of the technologies that are considered *safe to adopt* by the community. I.e. TypeScript has both a high satisfaction level and is widely used (cf. [State of JS: Satisfaction vs Usage](https://2020.stateofjs.com/en-US/technologies/#scatterplot_overview)).



![State of JS 2020: Satisfaction vs Usage](/assets/2022-06_state-of-js-satisfaction-vs-usage.png)



## Making type annotations part of JavaScript

### Motivation

Having established that static typing is generally useful for JavaScript development (and development in general), the question to address is this: why not just develop in TypeScript if we want to add static type checks to our JavaScript development process? 



One big argument in favor of types as comments in JavaScript compared to TypeScript (or similar languages and tools) is that it gives us the choice of type checker we want to use. We could for example opt for [Flow's](https://flow.org/) type checker instead of TypeScript's type checker if we preferred that (and if Flow started to support the types as comments syntax).



Another important argument pro types as comments in JavaScript is that it lets us skip the inevitable JavaScript transpilation step. Compare that to developing in TypeScript, where the usual workflow includes running our code through `tsc` (the TypeScript compiler) before executing it with `node` or in the browser. Dropping the transpilation step may significantly improve the speed of our development loop and moreover may require less set-up (such as the `tsconfig.json` file).

![Comparison of development workflow: type as comments and TypeScript](/assets/2022-06_types-as-comments-comparison.png)

> :information_source: A more elaborate discussion can be found here in [this article on the Microsoft dev blog](https://devblogs.microsoft.com/typescript/a-proposal-for-type-syntax-in-javascript/). (Picture above is also taken from the blog.)
> 



Lastly, even if we don't use type information for static type checking, it could be argued that they are a very good and less verbose replacement for [JSDoc](https://jsdoc.app/) style type comments. I.e., this

```javascript
/**
 * @param a {number}
 * @param b {number}
 */
function add(a, b) {
    return a + b;
}
```

simply would become this

```javascript
function add(a: number, b: number) {
  return a + b;
}
```

### What does it look like? 

The syntax of the [proposal for types as comments][tc39-types-as-comments] will probably not suprise anyone who has worked with either TypeScript or Flow. It rather seems like the authors aimed to capture the common subset of both. A lot of code written in TypeScript and Flow therefore already conforms to the proposed syntax.



Let's take a look at some examples to illustrate the similarities. 

#### Type Annotations



A **type annotation** is written in the form `x: T` where `x` is a variable name and `T` is a type name. We may use type annotations in variable declarations or in function parameters. 

```typescript
// a variable that stores a string
let x: string; 
// a function that takes two string parameters and returns a boolean value
function equals(s: string, t: string): boolean { /* … */ }
```

#### Type declarations



Types may be **declared** using the `interface` keyword to introduce a new type or using the `type` keyword to declare aliases.

```typescript
// An object type with a properties coordinate and radius.
interface Circle {
  coordinate: Point;
  radius: number
}
// A shorter alias for boolean
type bool = boolean;
```

Classes also implicitly define types.

```typescript
// Implicitly declares a type Point
class Point {
  x: number;
  y: number
}
```

#### Implicitly defined types

JavaScript's primitive data types have corresponding predefined types (such as `number` and `string`).



> :information_source: Curiously, the proposal suggests ignoring anything in between the curly braces of both `interface` as well as `class` declarations. This does not make much sense to me at the moment, but will hopefully be cleared up in the upcoming drafts.



#### Advanced features

Some more complicated type system features such as

* type unions (`type C = A | B`),
* generics (`type A<T> = T[]`, `interface Box<T> { content: T; }`)
* type and non-null assertions (`const point = JSON.parse('{"x": 0, "y": 0}') as Point` resp. `JSON.parse('{"x": 0, "y": 0}')!.x`)

are also proposed to be supported in order to add some more expressiveness. 

> :information_source: For the full list of features, take a look at [the current draft][tc39-types-as-comments].

#### Extensions to the module system

Lastly, an extension to the module system syntax is proposed so that types can be imported and exported as well using `import type` respectively `export type`.



### What the proposal is not about

When I first read through the proposal it strongly reminded me of the way optional type information was introduced via [type hints in Python](https://www.python.org/dev/peps/pep-0484/). I.e. adding type checking is entirely opt-in. You can use it if you want to, but you don't have to. It is not about introducing a specific type system. And, by extension, neither is it about standardising TypeScript's type system in the JavaScript specification.



This is a cautious and smart way to proceed on this topic in my opinion, because there is—as mentioned in the proposal—a very high risk that it would break the web. I.e. if web browsers would run type checks on JavaScript code, a lot of websites might break because they run type-wise unsound code. 

## Summary

The types as comments proposal laid out by the TC39 technical committee for JavaScript suggests adding syntax for optional typing information to JavaScript. With these syntax additions, we may provide type information in JavaScript code similar to comments. This type information may be used to  to leverage static type checking just as in TypeScript or Flow in order to improve code quality and safety. Having types as comments in JavaScript makes a lot of sense given the popularity of statically typed languages that compile to JavaScript such as TypeScript and Flow. We can make at least three arguments in favor of types as comments: 1. it gives us the choice of type checker, 2. we can prune transpilation from our development loop, and 3. type information is a less verbose replacement for JSDoc-style type comments.



Do you have any questions, suggestions, or comments? Get in touch with the author [via mail](https://aemail.com/Z4YQ)!

## About the Author: Friedrich Kurz

Friedrich has been working for MaibornWolff as a full-time software engineer for 2.5 years. In his current project, he's helping to build the AWS cloud infrastructure for a client's web platform. Friedrich considers himself a technology generalist with a broad range of interests rather than a technology specialist. He's, however, also very interested in functional programming and general methods of writing correct, clean, and maintainable code.

[tc39]: https://github.com/tc39
[tc39-types-as-comments]: https://github.com/tc39/proposal-type-annotations
[oreilly-programming-typescript]: https://www.google.de/books/edition/Programming_TypeScript/Y-mUDwAAQBAJ