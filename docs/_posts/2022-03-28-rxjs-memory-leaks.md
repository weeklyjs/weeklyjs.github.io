---
layout: post
title:  "Potential Memory Leaks in RxJS"
date:   2022-03-28 15:10:13 +0100
categories: typescript
author: Fabian Birke
---

## Why calling takeUntil() with only subject.next() leads to a new memory leak and how to fix it

RxJS is a JavaScript library for reactive programming, which makes it possible to map a complex asynchronous component flow in a comprehensible way. For this purpose, various states are kept within the application in the form of hot observables, which can be consumed at any time.

## Terminating 'hot' observables with takeUntil() - so far

Thus, a component for displaying data consumes the hot observables of a service and extends them with it's own logic. To properly terminate the component's observables, `takeUntil()` is called, which is triggered when the component is destroyed.

```typescript
@Component({  
selector: 'app-root',  
template: '<button (click)="cancelObservable()">Cancel Observable</button>',  
styleUrls: ['./app.component.scss']  
})
export  class  AppComponent  implements  OnInit, OnDestroy {

  public onDestroy$ = new BehaviorSubject<void>();  

  public result = new BehaviorSubject('1');  

  public result$ = this.result.pipe(  
    takeUntil(this.onDestroy$)  
  );  

  public cancelObservable() {  
    this.onDestroy$.next();  
  }  

  public ngOnInit(): void {  
    this.result$.subscribe({  
      next: () => console.log('result$ next()'),  
      complete: () => console.log('result$ complete()'),  
    });
  }
}
```

To demonstrate the problem, we have a component that tries to terminate its observables when a button is clicked. Normally the observable is called in the `OnDestroy` lifecycle method of the component.
The code works great and terminates the `result$` observable as soon as the `cancelObservable()` function is executed. To do this, `takeUntil()` is called with a subject in the observable pipe, with the subject emitting a value as soon as the component is destroyed. The fact that the observable is terminated can be seen by calling the `complete()` method, which is logged.

## How does takeUntil() work?

If you take a closer look at the implementation of `takeUntil()`, you can see that `takeUntil()` must subscribe to the subject that you put into the function. Only by subscribing, the function can know when to terminate the observable. But the subscribe triggers exactly the same problem that we tried to solve by the `takeUntil()`. Namely the avoidance of a memory leak.

```typescript
export function takeUntil<T>(notifier: ObservableInput<any>): MonoTypeOperatorFunction<T> {
  return operate((source, subscriber) => {
    innerFrom(notifier).subscribe(new OperatorSubscriber(subscriber, () => 		
    subscriber.complete(), noop));
    !subscriber.closed && source.subscribe(subscriber);
  });
}
```
  
Source: [Github: takeUntil()](https://github.com/ReactiveX/rxjs/blob/master/src/internal/operators/takeUntil.ts)
Now we have a situation where we have terminated the source observable, but have produced another leak as a result.

## The problem in more detail

To better demonstrate the problem, we implement a custom subject that prints a message to the console whenever `next()` or `complete()` is called on the subject.

```typescript
@Component({  
selector: 'app-root',  
template: '<button (click)="cancelObservable()">Cancel Observable</button>',  
styleUrls: ['./app.component.scss']  
})  
export class AppComponent {  

  public onDestroy$ = new AutoDestroySubject<void>();  

  public result = new BehaviorSubject('1');  

  public result$ = this.result.pipe(  
    takeUntil(this.onDestroy$)  
  );

  public cancelObservable() {  
    this.onDestroy$.next();  
    this.onDestroy$.complete();  
  }

  public ngOnInit(): void {  
    this.result$.subscribe({  
      next: () => console.log('result$ next()'),  
      complete: () => console.log('result$ complete()'),  
    }); 
  }
}  

export class AutoDestroySubject<T> extends Subject<T> {  
  constructor() {  
    super();  
  }
  
  public override next(value: T) {  
    console.log('AutoDestroySubject next() called');  
    super.next(value);  
  }

  public override complete() {  
    console.log('AutoDestroySubject complete() called');  
    super.complete();  
  }
}
```

Now we have implemented our own subject, which inherits from subject. To keep the functionality of the subject, we call `super()` in the constructor. We also override the functions `next()` and `complete()` and log something into the console to demonstrate what happens. 

If you click on the button now, you will see in the console that `AutoDestroySubject next() called` is displayed. However, not that `complete()` has been called. This shows us that there is still an observable running and we still have a memory leak.

## How do you manage to have the subject terminate after the .next()?

The answer to this is simple. Just call `subject.complete()`.

```typescript
public cancelObservable() {  
  this.onDestroy$.next();  
  this.onDestroy$.complete();  
}  
```

If we add the `complete()` to the `cancelObservable()` method and click on the button, then the following entries appear in the console: `AutoDestroySubject next() called` and `AutoDestroySubject complete() called` . Which in turn means that we have resolved all MemoryLeaks.

## Extend the AutoDestroySubject to close it directly after the next().

```typescript
export class AutoDestroySubject<T> extends Subject<T> {  
  constructor() {  
    super();  
  }
  
  public override next(value: T) {  
    super.next(value);  
    super.complete();  
  }
}
```     
 
By calling `super.complete()` directly in the `next()` method of the subject after the first value is emitted, we ensure that the memory leak of the subject is also resolved as soon as a value is emitted.

At the end we delete the line with the `this.onDestroy$.complete();` from the `cancelObservable` method. If we now click on the button, all memory leaks will be resolved.

Instead of the simple Subject, it is recommended to use the `AutoDestroySubject` everywhere in the application, which has the sole task to emit a value once before it closes automatically.

## Naming of the Subject

Last but not least, we can think about the name of the subject. Currently we have chosen the name `AutoDestroySubject`, which describes the purpose, i.e. *the automatic destruction of things*. However, it would also be an option to describe the exact functionality in the name. 

For example with the name `SingleEmitSubject`. So everyone knows that this subject has a special feature, namely that only one value is emitted.

## About the author: Fabian Birke

Fabian joined MaibornWolff GmbH in 2019 and has several years of experience as a full-stack developer. Currently he focuses on the development of dynamic frontend applications.

