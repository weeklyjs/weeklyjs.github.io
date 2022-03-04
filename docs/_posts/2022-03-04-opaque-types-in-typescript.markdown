---
layout: post
title:  "Opaque Types in TypeScript"
date:   2022-03-04 14:00:13 +0100
categories: typescript
author: Raphael Pigulla
---

TypeScript already gives us a lot in terms of type safety - many types of bugs that plagues us in the days of Vanilla JavaScript are a thing of the past. But we can go even further by leveraging the type system in clever ways.

Let's say we have some sort of controller that assigns a todo to a user, referencing both via their respective id:
```typescript
@Post()
public assignTodoToUser(
    @Query('userId') userId: number,
    @Query('todoId') todoId: number,
): void {
    this.todoService.assignToUser(userId, todoId)
}
```
A potential issue here is that the `userId` and `todoId` are both `numbers` so that one could easily swap them, maybe while doing some refactoring, and thus introduce a bug that might not be so easy to spot:
```typescript
this.todoService.assignToUser(userId, todoId)
// compiles just as fine as this:
this.todoService.assignToUser(todoId, userId)
```

The reason for this is that types are _transparent_ by default, meaning that structrually identical types are interchangeable. _Opaque types_, by contrast, are not:
```typescript
type UserId = Opaque<number>

// This will not compile - the types are incompatible.
const userId: UserID = 42

// You need to cast the type explicitly instead:
const userId = 42 as UserID
```
We are using the [`type-fest`](https://github.com/sindresorhus/type-fest) library here, but there isn't really much to it. You can read up on some of the details in the [excellent article](https://codemix.com/opaque-types-in-javascript/) by Charles Pick.

With this change, our example from above is now much more robust (assuming the `assignToUser` method types its parameters accordingly):
```typescript
@Post()
public assignTodoToUser(
    // In this particular case we don't even need to cast!
    @Query('userId') userId: UserID,
    @Query('todoId') todoId: TodoID,
): void {
    this.todoService.assignToUser(userId, todoId)
}
```
If the arguments are swapped, the TypeScript compiler will complain!

It is possible to go even further here and delegate the conversion to these types to dedicated methods:
```typescript
type UserID = Opaque<number>

function asUserID(value: number): UserID {
    if (!Number.isInteger(value) || value <= 0) {
        throw new TypeError('Not a valid user id');
    }

    return value as UserID;
}
```
You can be sure to always have a proper user id wherever you see a `UserID` - as long as you can resist the temptation to just do `someValue as UserID`. (Maybe a custom linting rule could help with that.)

In any case, using opaque types for ids is quick win with virtually no drawbacks: the only time you should ever need to convert values is when they enter your application, typically when you receive data in a controller or read it from the database!

About the author: Raphael Pigulla

Raphael has more than 10 years of experience in software development. He specializes in backend development with Node.js and TypeScript. Raphael joined [MaibornWolff](https://maibornwolff.de) in 2019.
