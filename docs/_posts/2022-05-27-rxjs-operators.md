---
layout: post
title:  "Avoid canceled requests when using RxJS combineLatest()"
date:   2022-05-27 06:00:13 +0100
categories: angular
author: Fabian Birke
---

If you work with Angular in your project, then you can't avoid the reactive framework RxJS. This offers a lot of of operators. While some are rarely used, `combinedLatest` is certainly one of the great favorites of any Angular developer. It is used to combine multiple observable streams. It emits a value as soon as all incoming observables emit at least one value.

## Example with backend request
Our example is quite simple. We have two observables: day and month. These are combined with `combineLatest`. As soon as the combined observable changes, a request is executed against a backend. The request is in a `switchMap()`. In the UI we have three buttons to change the day, the month and both at the same time (day + month).

```typescript

  public day$ = new BehaviorSubject(0);
  public dayCount = 0;
  
  public month$ = new BehaviorSubject(0)
  public monthCount = 0;

  public result$ = combineLatest([this.day$, this.month$]).pipe(
    debounceTime(0),
    switchMap( ([day, month]) => this.dataService.apiCall(day, month))
  )

  increaseDay() {
    this.dayCount++;
    this.day$.next(this.dayCount);
  }

  increaseMonth() {
    this.monthCount++;
    this.month$.next(this.monthCount);
  }

  increaseDayAndMonth() {
    this.increaseDay();
    this.increaseMonth();
  }
```

If you click on the button in the UI to trigger `changeDay()`, a request is sent to the backend. With `increaseMonth()`
it is the same. So far everything is fine.

But if we now use the method `increaseDayAndMonth()` to change both at the same time, i.e. day and month,
then we see that a canceled request is displayed in our console.

The request is canceled because we use `switchMap`, which ensures that only the most recent request is active.

So the problem is that in a very short time our incoming observables are changing and therefore triggering the backend request twice in a row.

## Solution
To avoid the problem, the `debounceTime` operator with the value `0` can be used. The value `0` means that 0 milliseconds are waited until the emitted values pass through.

However, it also says that this will only happen at the end of the call stack. This can be considered as a kind of loop, only when the loop has run through, the request is 
executed. Since within the loop is not only the change of "days", but also the change of "month", the request is executed only once.

## About the author: Fabian Birke

Fabian joined MaibornWolff GmbH in 2019 and has several years of experience as a full-stack developer. Currently he focuses on the development of dynamic frontend applications.
