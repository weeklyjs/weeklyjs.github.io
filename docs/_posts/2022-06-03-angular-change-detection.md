# AngularChangedetectionPlayground

some colleagues who have not yet dealt much with the topic of detection in Angular may find it difficult to imagine what the difference is.

Here I would like to compare a few values.


## What is ChangeDetection?

ChangeDetection in Angular is a process, which check if components have to rerender.

By Default, each component is permanently checked whether values have changed.

In my main project which grow over the years it is nearly endless processing.

So the idea was, step by step migrate to onPush.

But why the difference?

With OnPush, with the available board tools and yourself, it's up to you to trigger the detection.

In the next sections I hope I serve you a better overview what happend and why it is important to not ignore that topic.


## Before to start

what is the difference between `markForCheck` and `detectChanges`?

### markForCheck

`markForCheck` should always used if you have to signal the detector to check that view.

### detectChanges

`detectChanges` should used to check the view and them children.

In combination with `detach` it's able to implement local change detection checks.


> Warning: detach components should be done with caution!

## Expectation

Default: Angular plays it safe and rather checks once too often than too little if something has changed and renders the DOM of the component again. This is done throughout the DOM tree.

OnPush: Angular relies on the ChangeDetection base board means to be sufficient, or pipes and components to inform the ChangeDetectionRef if something has changed outside the board means.

## Setup

Angular ~13

Material ~13

AppComponent - the App Wrapper

DetectionBlockComponent - Components which create Recursive Child-Components.

## Playground

my Component tree looks like:

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

the following steps I clicked in the appliation:

1. Open Page
2. click detectChanges 4
3. click markForCheck 7
4. click markForCheck 4
5. click detectChanges 2
6. click markForCheck App
7. click detectChanges App


# Test results

## app default

when app is on default(and components on push), we come to the following numbers

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

### explaination

initial the complete tree will be check from the detector.

after the initial check, the app component in default mode will check multiple times itself and its children.

when `detectChanges` a node in the tree, itself and its children will check. also, when it isn't detached, the whole tree until there.

and again, default, app will check them children multiple times.

`markForCheck` will just check itself and then the tree. so the child of 7 will check while the tree check not direct. that can be important in some cases!


## app onpush

when app is onPush, we come to the following numbers

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

### explaination

wow! just app component onPush v.s. default made round about 45%!

in initial check, 1 and 2 won't check twice, only app still check multiple times thich that setup.

also additional steps won't check less times.

App is more stable and won't check the tree everytime so aggressive.

What's the pitfall here? Mutations in objects without trigger detection by yourself can occure stale views. So be careful how you write component logic. Best practices IMHO are observables with async subscription in the template or call `markForCheck` after async processes in the component.

## detach components after view init

> this section is more experimental!
> In this case, the simple application setup made it possible receive the same application behavior then before.
> Larger apps may get more stale and shaky when detach without caution!

with detach the components after intial rendering the following numbers


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


### explaination

awesome, from 83 to 38 is a lot of circles which we safed!

what's the reason?

by detaching all components from the change detection, you have to full control the detection of them.

for example, after initial the `detectChanges` of 4 will only effect app, 4 and 5.

5 is the child, 4 itself and app because it's the root node and have to check how much of the total tree (which nomore exists more or less) has to check like before.

the multiple times of checking app in the next steps is for me also a secret, I'll check them later.

What are the risks with this setup?

1. Detach can hide changes in the detector circles, so changes in your component won't notify the renderer and the DOM will be stale.
2. You have to do a lot of things manually, like detection and detach(reattach) the component
3. child components may not render or act correct. In my first experiments MatButtons or MatInputs won't display correct.

## my learning while writing this post

1. detach static "leaf" components can safe performance 
2. from default to onpush can be hard but is essential for larger applications


### numbers behind the tests

see [here](../assets/numbers.md)
