---
layout: post
title: "Angular Changedetection Playground"
date: 2022-06-03 06:00:13 +0100
categories: angular
author: Nils Heinemann
---

Some colleagues who have not yet dealt much with the topic of ChangeDetection in Angular may find it difficult to imagine what the 
difference between the default and the `OnPush` approach is.

When your application gets larger this topic gets more relevant tou you.

Here I would like to compare and show how ChangeDetection can have an effect on the performance of an application.

## What is ChangeDetection?

ChangeDetection in Angular is a process which check if components have to rerender.

By Default, each component is permanently checked whether values have changed.

In my main project which grew over the years it is a nearly endless type of processing.

So the idea was, step by step migrate to `OnPush`.

But what is the difference?

With `OnPush` it's up to you to trigger the ChangeDection yourself, using the available tools.

In this articls I hope to give you a better overview of the differences and why it is important to not ignore that topic.

## Before to start

what is the difference between `markForCheck` and `detectChanges`?

### markForCheck

`markForCheck` should always be used if you have to signal the detector to check that view.

### detectChanges

`detectChanges` should be used to check the view and the children of it.

If you want to check only a small scope of your detection tree, use `detach`.

> Warning: detach components should be done with caution!

## Expectation

Default: Angular will rather check the DOM tree too many times than not enough.

OnPush: In Angular it's your own responsibility to use build-in capabilities like Pipes, Components and the ChangeDetectorRef to define where checks are necessary outside the lifecycle routines.

## Setup

Angular ~13

Material ~13

Now I going to use the following two components to build my test angular component tree.

AppComponent - the App Wrapper

DetectionBlockComponent - Components which create recursive Child Components.

## Playground

My Component tree looks like:

```markdown
| App
| - 1
|   - 3
|     - 4
|       - 5
|         - 6
| - 2
|   - 7
|     - 8
```

## Test Process

The following steps I clicked in the appliation:

1. Open Page
2. click detectChanges 4
3. click markForCheck 7
4. click markForCheck 4
5. click detectChanges 2
6. click markForCheck App
7. click detectChanges App


# Test results

## app in default detection

We come to the following numbers

| Step | check cycles |
|---|---|
| 1 | 15 |
| 2 | 13 |
| 3 | 12 |
| 4 | 13 | 
| 5 | 11 |
| 6 | 10 |
| 7 | 9 |
| total | 83 |

### Explaination

the complete tree will be check from the detector on initial.

After the initial check, the app component in default mode will check multiple times itself and the children of it.

When click on `detectChanges` of a node in the tree, itself and its children will be checked. If you don't use `detach` the ChangeDetection will start at the top of the tree and check until the origin of the detection event.

And again, default, app will check their children multiple times.

What happens with `markForCheck`:

1. checks 7
2. checks app
3. checks 1
4. checks 2
5. checks 7
6. checks 8
7. double checks app again

## app in onpush detection

We come to the following numbers

| Step | check cycles |
|---|---|
| 1 | 11 |
| 2 | 9 |
| 3 | 8 |
| 4 | 9 |
| 5 | 7 |
| 6 | 6 |
| 7 | 5 |
| total | 55 |

### Explaination

Wow! Just app component onPush v.s. default performed better by about 45%!

In initial check, '1' and '2' won't check twice, only app still check multiple times in that setup.

Also additional steps won't check as often.

App is more stable and won't check the tree everytime so "aggressive".

What's the pitfall here? Mutations in objects without trigger detection by yourself can occure stale views. So be careful how you write component logic. Best practices IMHO are observables with async subscription in the template or call `markForCheck` after async processes in the component.

## detach components after view init

> This section is more experimental!
> In this case, the simple application setup made it possible to receive the same application behavior then before.
> Larger apps may get more stale and shaky when detach without caution!

When you have call `detach` the components after intial rendering show the following numbers

| Step | check cycles |
|---|---|
| 1 | 11 |
| 2 | 5 |
| 3 | 4 |
| 4 | 4 |
| 5 | 5 |
| 6 | 4 |
| 7 | 5 |
| total | 38 |


### Explaination

Awesome, from 83 to 38 is a lot of circles which we safed!

What's the reason?

By detaching all components from the ChangeDetection, you take over the full control over the ChangeDetection of them.

For example, after 'initial' step the `detectChanges` of '4' will only effect app, '4' and '5'.

'5' is the child, '4' itself and app because it's the root node and have to check how much of the total tree (which nomore exists more or less) has to check like before.

> The reason for multiple checks of app is not clear to me, I'll check it later.

What are the risks with this setup?

1. Detach can hide changes in the detector circles, so changes in your component won't notify the renderer and the DOM becomes stale.
2. You have to do a lot of things manually, like detection and detach(reattach) the component
3. Child components may not render or act correctly. In my first experiments with `MatButton` or `MatInput` it won't display correct.

## My learning while writing this post

1. `detach` static "leaf" components can safe performance 
2. From `Default` to `OnPush` can be hard but is essential for larger applications


### Numbers behind the tests

See [here](../assets/numbers.md)


## About the author: Nils Heinemann

Nils has more than five years of experience in software development and architecture. He specializes in frontend development with Angular and TypeScript. Nils joined MaibornWolff in 2018.
