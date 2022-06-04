---
layout: post
title: "Angular change detection Playground"
date: 2022-06-03 06:00:13 +0100
categories: angular
author: Nils Heinemann
---

Some colleagues who have not yet dealt much with the topic of change detection in Angular may find it difficult to imagine what the difference between the default and the `onPush` approach is.

When your application gets larger this topic gets more relevant to you.

Here I would like to compare and show how change detection can affect the performance of an application.

## What is change detection?

Change detection in Angular is a process that checks if components have to rerender.

By Default, each component is permanently checked whether values have changed.

In my main project which grew over the years, it is a nearly endless type of processing.

So the idea was to migrate to `OnPush` step by step.

But what is the difference?

With `OnPush` it's up to you to trigger the ChangeDection yourself, using the available tools.

In this article, I hope to give you a better overview of the differences and why it is important to not ignore that topic.

## Before we start

what is the difference between `markForCheck` and `detectChanges`?

### `markForCheck`

`markForCheck` should always be used if you have to signal the detector to check that view.

### detectChanges

`detectChanges` should be used to check the view and the children of it.

If you want to check only a small scope of your detection tree, use `detach`.

> Warning: detach components should be done with caution!

## Expectation

Default: Angular will rather check the DOM tree too many times than not enough.

OnPush: In Angular, it's your responsibility to use build-in capabilities like Pipes, Components, and the ChangeDetectorRef to define where checks are necessary outside the lifecycle routines.

## Setup

Angular ~13

Material ~13

Now I going to use the following two components to build my test angular component tree.

AppComponent - the App Wrapper

DetectionBlockComponent - Components that create recursive Child Components.

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

I performed the following steps in the application:

1. Open Page
2. click `detectChanges` 4
3. click `markForCheck` 7
4. click `markForCheck` 4
5. click `detectChanges` 2
6. click `markForCheck` App
7. click `detectChanges` App


# Test results

## What does our app do during default detection?

We obtain the following numbers:

| Step  | check cycles |
| ----- | ------------ |
| 1     | 15           |
| 2     | 13           |
| 3     | 12           |
| 4     | 13           |
| 5     | 11           |
| 6     | 10           |
| 7     | 9            |
| total | 83           |

### Explanation

The complete tree will be checked by the detector on the initial check.

After the initial check, the app component in default mode will check multiple times itself and the children of it.

When we click on `detectChanges` of a node in the tree, itself and its children will be checked. If you don't use `detach` the change detection will start at the top of the tree and check until the origin of the detection event.

And again, by default, the app will check its children multiple times.

What happens with `markForCheck`:

1. checks 7
2. checks app
3. checks 1
4. checks 2
5. checks 7
6. checks 8
7. double checks app again

## What does our app do during `onPush` detection?

We obtain the following numbers:

| Step  | check cycles |
| ----- | ------------ |
| 1     | 11           |
| 2     | 9            |
| 3     | 8            |
| 4     | 9            |
| 5     | 7            |
| 6     | 6            |
| 7     | 5            |
| total | 55           |

### Explanation

Wow! As we can see, `onPush` detection performs about 45% better than default detection!

During the initial check, '1' and '2' won't check twice, only the app will still be checked multiple times in that setup.

Also, additional steps won't be checked as often.

Additionally, the app is more stable and won't check the tree every time.

What's the pitfall here? Mutations in objects without trigger detection by yourself can produce stale views. So, we have to be careful when we write component logic. Best practices IMHO are observables with async subscription in the template or call `markForCheck` after async processes in the component.

## Detach components after view init

> This section is more experimental!
> In this case, the simple application setup made it possible to receive the same application behavior as before.
> Larger apps may get more stale and shaky when detach without caution!

When you have called `detach` the components after initial rendering, this will produce the following numbers

| Step  | check cycles |
| ----- | ------------ |
| 1     | 11           |
| 2     | 5            |
| 3     | 4            |
| 4     | 4            |
| 5     | 5            |
| 6     | 4            |
| 7     | 5            |
| total | 38           |


### Explanation

Awesome, from 83 to 38 is a lot of circles that we saved!

What's the reason?

By detaching all components from the change detection, you take over the full control over the change detection of them.

For example, after 'initial' step the `detectChanges` of '4' will only effect app, '4' and '5'.

'5' is the child, '4' itself and the app because it's the root node and has to check how much of the total tree (which no more exists more or less) has to check like before.

> The reason for multiple checks of the app is not clear to me, I'll check it later.

What are the risks with this setup?

1. Detach can hide changes in the detector circles, so changes in your component won't notify the renderer and the DOM becomes stale.
2. You have to do a lot of things manually, like detection and detach(reattach) the component
3. Child components may not render or act correctly. In my first experiments with `MatButton` or `MatInput` it won't display correctly.

## My learning while writing this post

1. `detach` static "leaf" components can save performance 
2. From `Default` to `OnPush` can be hard but is essential for larger applications


### Numbers behind the tests

See [here](../assets/numbers.md)


## About the author: Nils Heinemann

Nils has more than five years of experience in software development and architecture. He specializes in frontend development with Angular and TypeScript. Nils joined MaibornWolff in 2018.
