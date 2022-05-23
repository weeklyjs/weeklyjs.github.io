---
layout: post
title:  "Handling Conditional Classes in Angular"
date:   2022-05-20 14:00:13 +0100
categories: angular
author: Nils Heinemann
---

How do you handle conditional classes at a dom element in Angular? There are two main options: class-bindings and `ngclass`. But which one is better?
I tested both to get a definite answer.

## ngclass vs class-binding

How to use `ngclass`?

```html
<div [ngClass]="{
  class0: condition,
  classN: condition
}"></div>
```

And how would class-binding look like?

```html
<div [class.class0]="condition"
     [class.classN]="condition"
></div>
```

## Test Scenario

### Project Structure

```
app.component.class.html => contains class-binding version of the template

app.component.ngclass.html => contains ngclass version of the template

app.component.css => basic css to provide the demo styles

app.component.ts => align state and callbacks for the buttons to change the alignment

index.html => basic resetted index file
```

### Do it yourself

just clone the [repository](https://github.com/SourceCodeBot/angular-class-binding-demo) and execute `npm start`.

on http://localhost:4200 you can directly interact with the current selected version.

### Test Plan

1. Open the Devtools
2. Open the [angular profiler plugin](https://chrome.google.com/webstore/detail/angular-devtools/ienfalfjdbdpebioblfackkekamfmbnh)
3. switch to "profiler" tab
4. start recording by click the round button which become red by hover
5. press 
   1. top
   2. bottom
   3. left
   4. right
   5. center
6. click "safe profile" to export a json with the detail report

## My Result

> fnumbers in seconds

| Direction | ngclass | Class-binding |
| :-------: | :-----: | :-----------: |
|    top    | 0,4999  |    0,2999     |
|  bottom   | 0,7000  |    0,4000     |
|   left    | 0,6000  |    0,4000     |
|   right   | 0,3999  |    0,3000     |
|  Center   | 0,4000  |    0,0999     |


## Recommandation

To avoid complexity in your change detection circle you should probably use class-binding instead of `ngclass`.

For more complex usecases or if you want to bind a object with the typesignature `Record<string, boolean>` `ngclass` would be the right way.

## About the author: Nils Heinemann

Nils has more than five years of experience in software development and architecture. He specializes in frontend development with Angular and TypeScript. Nils joined MaibornWolff in 2018. If you want to work with Nils, we are looking for [Web Developers](https://www.maibornwolff.de/en/careers/job-vacancies/web-developer)!
