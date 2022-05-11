---
layout: post
title:  "Best Practices FP in TypeScript"
date:   2022-04-08 10:00:13 +0100
categories: typescript
author: Nils Heinemann
---

Some of us have a background in functional programming and are now moving into a TypeScript environment. 

If you are part of this group, you might be wondering how much FP is possible in TypeScript.

Bad news first: not as much as you might like. However, some things will work. We'll take a look at function abstraction and currying for compose.

## Simple function abstraction

Some anomynous functions can be extracted very simple:

```typescript
// query dto from api and transform into ui-model
class ApiService {
  public query(filter = ''): Promise<UiModel | null> {
    return fetch(`${url}?filter=${filter}`).then(mapIntoUiModel).catch(() => null);
  }
}

function mapIntoUiModel(response: ResponseData): UiModel {
  return // magic here
}
```

Also Array transform functions can extract abstract to utils:

```typescript
export function extractValuesByKey<T, K extends keyof T>(key: K): (item: T) => T[K] {
  return (item) => item[key];
}

// usage: list.map(extractValuesByKey('prop'))
```
Nothing complicated here, right?

## currying for compose

But there is more. In FP it's sometimes a good practices to build new functions by composing. Compose is based on a simple law:

![Compose in FP](/assets/fp_2022_08_04.png)

In Typescript it would look like this:

```typescript
type ComposeFn = <A, B, C>(
  f: (a: A) => B
) => (g: (b: B) => C) => (a: A) => C;

// g o f => g(f(arg))
export const compose: ComposeFn = <A, B, C>(f: (a: A) => B) => (g: (b: B) => C) => (arg: A) => g(f(arg));
```

Let's look at a small example now:

```typescript
// sort items
function sortFn(items: Model[]): Model[] {
  return items.sort();
}

// filter items
function filterFn(fn: (item: Model) => boolean): (items: Model[]) => Model[] {
  return (items) => items.filter(fn);
}

const filter = filterFn((item) => item.name.startsWith('A'));

// items -> sort -> filter
const filterAndSort = compose(filter)(sortFn);

const items: Model[];

const finalData = filterAndSort(items);

```

That Functions return functions can also be useful for other use cases, in this example with Cypress:

```typescript 
function baseApiFn(baseUrl: string): (path: string) => string {
  return (path) => `${baseUrl}${path}`;
}

describe('mySuite', () => {
  const baseUrlFn = baseApiFn(Cypress.config().baseUrl);
  
  it('should ...', () => {
    const userApi = baseUrlFn('/user');
    cy.intercept(userApi).as('user');
    // do something
    cy.wait('@user');
  });
  
  it('should ... something other', () => {
    const authApi = baseUrlFn('/token');
    cy.intercept(authApi).as('login');
    // do something
    cy.wait('@login');
  });
  
});
```

We built a function `baseApiFn` to manage a preloaded `baseUrl` value and evaluate the final result on demand.

In a more advanced variant of this we additionally replace path variable:

```typescript
type ParameterBaseUrlFn = (
  subPath?: string
) => (parameters?: Record<string, string>) => string;

function baseApiFn(baseUrl: string): (path: string) => string {
  return (path) => `${baseUrl}${path}`;
}

export function baseUrlFnWithPathVariables(
  baseUrl: string
): ParameterBaseUrlFn {
  const baseFn = baseApiFn(baseUrl);
  return (subPath) => parameters =>
    Object.entries(parameters ?? {}).reduce((url, [key, value]) => {
      return url.replace(`{${key}}`, value);
    }, baseFn(subPath));
}
```

This is how you can use currying for compose in TypeScript.

## About the author: Nils Heinemann

Nils has more than five years of experience in software development and architecture. He specializes in frontend development with Angular and TypeScript. Nils joined MaibornWolff in 2018.
