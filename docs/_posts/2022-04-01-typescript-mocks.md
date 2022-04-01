---
layout: post
title:  "Typesafe Mocks in TypeScript"
date:   2022-04-01 17:00:13 +0100
categories: typescript
author: Nils Heinemann
---

Mocks are great, even if they represent the same API as the real implementation. In this article we use global mocks for implementations and won't mock them multiple times.

2 small utils in tyepscript have solved this problem for me in the past.

## Public

`Public<T>` is a great helper to make sure that my mocks will match the API of the real implementation.

```typescript
export type Public<T> = Pick<T, keyof T>;
```

It will pick each public property and method of a class.

## injectMock

In my daily work with Angular and TestBed, I have written this ugly Boilerplate multiple times:

```typescript
const service = (TestBed.inject(RealService) as unknown) as MockService;
```

Now, let's take a look at `injectMock`.

While I declarate my mock like this:

```typescript

class MockService implements Public<RealService> {
   public realProp: string = 'hello World'; // prop from RealService
   public printString() {
      console.log(this.realProp); // exists only in MockService
   }
}

export const injectMockService = injectMock<RealService, MockService>(RealService);

```

and `injectMock` looks like:

```typescript
export function injectMock<T, M extends Public<T>>(
  type: Type<T>
): () => T) => M {
  return () => (TestBed.inject(type) as unknown) as M;
}
```

and the usage in my spec will now be done like this:

```typescript
const mockService = injectMockService(); // now safe in type MockService
mockService.realProp; // work fine
mockService.printString(); // work also
```

It's not the breaking change in development, but it will save some time while writing reusable mocks.

## About the author: Nils Heinemann

Nils has more than five years of experience in software development and architecture. He specializes in frontend development with Angular and TypeScript. Nils joined [MaibornWolff](https://www.maibornwolff.de/) in 2018.