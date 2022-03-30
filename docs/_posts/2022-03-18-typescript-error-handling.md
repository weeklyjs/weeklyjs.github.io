---
layout: post
title:  "TypeScript Error Handling"
date:   2022-03-18 09:00:13 +0100
categories: typescript
author: Tim-Lukas Maeke
---

TypeScript offers many ways to handle HTTP errors. I will show you a way that works for me.
In my current project we are building a server-side NestJS backend for a web application, which is the use case we'll look into now.

When you want to handle a HTTP error in this kind of setup you first need to use `try-catch` to wrap your method.
In my example I just want to make a `get` request to a website and when the request fails I'll log the error to the console.

```typescript
try {
  return await lastValueFrom(
    this.http.get(`www.example.com`).pipe(map((response) => response.data))
  );
} catch (error: unknown) {
  console.log(error);
}
```

This approach looks plausible at first, but on closer inspection `console.log` is not a proper way to handle an http error in a live system.
It's pretty simple: These errors are never thrown and you will not recongnise them in your live system.
The simplest version of handling errors would be to just throw them like this:

```typescript
catch (error: unknown) {
    throw new UnauthorizedException();
}
```

Now, what if the error isn't `unauthorized` but `NotFoundException`?
Then you will always receive an `unauthorized exception` and you will never know what the exact error is.

A better way to handle errors is to use a handler function which decides what kind of error it is and what you want to do with this error.

The first step is to hand over the error you receive from the `catch` block to the `handle` function:

```typescript
export function handleError(error: unknown): never {
    switch(error.response?.status) {
        case HttpStatus.UNAUTHORIZED:
            throw new UnauthorizedException(error.message);
        case HttpStatus.NOT_FOUND:
            throw new NotFoundException(error.message);
        case HttpStatus.INTERNAL_SERVER_ERROR:
        default:
            throw new InternalServerErrorException(error.message);
    }
}
```

In the `switch` message you can add all errors you want to throw. We'll assume that yo do not want to throw any other other errors than these.
If an other error occures it is automatically an `InternalServerError`, because this is the default we specified.

After we finished the handler function just replace the `console.log` with the `handleError` function.

```typescript
} catch (error: unknown) {
    handleError(error);
}
```

When you now call the get function and receive an error your handleError function will throw one of the three specified errors.

**About the author**

Tim is from Hamburg where he studied at the University of Applied Sciences. He loves cycling around Hamburg and in the whole north of Germany. Since 2021 he is working as a Software Engineer at MaibornWolff. Tim has several years of experience in software development. He specializes in backend development with Node.js and TypeScript.

