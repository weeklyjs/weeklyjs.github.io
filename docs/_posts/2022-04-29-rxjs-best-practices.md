---
layout: post
title:  "RxJS Best Practices and an Idea"
date:   2022-04-29 08:00:13 +0100
categories: javascript
author: Nils Heinemann
---

There are some best practices in RxJS for several regular use cases.
First of all I'll share my personal recommandations from my long time project, before we go into an idea how to improve them.

## Clean flow

Nothing is more important for my colleageues and me than a clean flow of operators in a pipe block:

```typescript
public userName$ = this.loginService.onLogin$.pipe(
  switchMap((credentials) => this.authService.authenticate(credentials)),
  tap((user) => !user && this.router.navigate(['/login'])),
  filterNotNull(), // custom
  map(({userName}) => userName),
  share({
    resetOnRefCountZero: true,
    connector: () => new ReplaySubject(1)
  })
);
```

However, is this really clean? Yes, it somehow is, but `tap` isn't a great choice in this position, as side effects in a pipe flow should be avoided as much as possible.
What would that look like in a "bad" case?

```typescript
public userName$ = this.loginService.onLogin$.pipe(
  switchMap((credentials) => this.authService.authenticate(credentials).pipe(
    tap((user) => !user && this.router.navigate(['/login'])),
    filterNotNull(), // custom
  )),
  map(({userName}) => userName),
  share({
    resetOnRefCountZero: true,
    connector: () => new ReplaySubject(1)
  })
);
```

"Bad" is too much of a harsh word for this example, but I hope it gives a rough overview of the problem.
**Important**: in case of errors it could sometimes be necessary to nest some pipes. Keep an eye on that!

## Custom operators

Have you ever seen something like this?

```typescript
....pipe(
  filter((dtoOrNull) => !dtoOrNull),
  map((dto) => dto!.prop)
)
...
```

Looks familiar, right? Yes, but a simple TypeScript trick and RxJS operator can fix that.

```typescript
export function filterNotNull<T>(): OperatorFunction<T, Exclude<T, null | undefined>> {
  return value$ =>
    value$.pipe(filter(value => value != null)) as Observable<Exclude<T, null | undefined>>;
}
```

Why is that casting necessary? `filter<T>` is a mono type operator function that mean Input = Output type.
If you see multiple combinations of operators in several pipes, you can just extract them into one custom pipe. Try it out!

## OperatorFunctions from Services

Now, let's move on from these known best practices to a new idea: this short sample should give a brief overview.

```typescript
@Injectable()
export class FilterService {
  private termSubject = new BehaviorSubject('');
  
  public currentTerm$ = this.termSubject.asObservable();
  
  public filterFn<T>(matchFn: (item: T, term: string) => boolean)): MonoTypeOperatorFunction<T[]> {
    return source$ => combineLatest({items: source$, term: this.currentTerm$}).pipe(
    	map(({items, term}) => items.filter(item => matchFn(item, term)))
    );
  }
  
  public setFilterTerm(term: string): void {
    this.termSubject.next(term);
  }
}

@Injectable()
export class ListService {
  public listItems$ = this.apiService.listData$.pipe(
    this.filterService.filterFn<ListItem>((item, term) => item.title.contains(term))
  );
  
  constructor(private filterService: FilterService) {}
}
```

Basically it directly emits the observable at the point where we use the service function in our pipe, we don't need to combine something anymore or do anything else. Our `FilterService` can be used in our filter field and every time if the component emit a new term to the service, the `listItems$` will update. many other useacses for this are conceivable.

## About the author: Nils Heinemann

Nils has more than five years of experience in software development and architecture. He specializes in frontend development with Angular and TypeScript. Nils joined MaibornWolff in 2018. If you want to work with Nils, we are looking for [Web Developers](https://www.maibornwolff.de/en/careers/job-vacancies/web-developer)!
