---
layout: post
title:  "Why you can't trust TypeScript Interfaces"
date:   2022-03-11 14:00:13 +0100
categories: typescript
author: Michael Hammerl
---

TypeScript usually does a pretty good job in detecting bugs already at compile time. 
However, sometimes it can also introduce problems that developers wouldn't even think about, especially if they are used to naturally typed languages.

One such weird behavior can be found in the usage of interfaces. Even though they only exist pre-compile time, they can behave very unexpectedly.

## Covariance & Contravariance

Before we dive into an example, let's have a quick look at the concepts of **covariance** & **contravariance**.

Let's assume that we want to define an interface
which takes a `User` or user-id (`string`) and returns their `Address` (or `null` if nothing was found).

```typescript
    interface IAddressRepository {
        getAddress(user: User | string): Address | null;
    }
```    

During the implementation of that interface, it is perfectly fine to return a sub-type of `Address | null`, in this case an `Address`. 
This is called **covariance**.


```typescript
    getAddress(user: User | string) { return new Address };     // Ok
    getAddress(user: User | string) { return {} };              // Not acceptable
```  

Method parameters, on the other hand, are usually **contravariant**. 
It is perfectly fine to accept more types - as long as the required type is included.

```typescript
    getAddress(user: User | string | number) { return new Address };    // Accepting also a number is fine
    getAddress(user: User) { return new Address };                       // This doesn't handle strings as required by the interface
``` 

Actually, the last example will be accepted by the TypeScript compiler as a valid implementation of our interface.
This is where it becomes a bit confusing.

Function parameters are treated contravariantly as long as the `strict` or `strictFunctionTypes` flag is set to `true`.
Method parameters, however, mostly use **bivariance**. That means, it accepts both sub- and super-types.

This can lead to wrong implementations of interfaces without being detected by TypeScript. Let's look again at our example:

```typescript
    interface User { userId: string; }
    interface Address { street: string; }

    interface IAddressRepository {
        getAddress(user: User | string): Address | null;
    }
    
    class AddressRepository implements IAddressRepository {
        getAddress(user: User) { console.log(user.userId.length); return null; }; 
    }
    
    const repository: IAddressRepository = new AddressRepository();
    repository.getAddress('someUserID');
``` 

This will not produce any TypeScript errors. 
Only at runtime you will receive an error, since the `getAddress` implementation can only deal with `User`objects.

## Solution

There is another way to define our interface.
If we instead use the function signature, our method parameters will be treated contravariantly instead:

```typescript
    interface IAddressRepository {
        getAddress(user: User | string): Address | null;        // bivariance (method signature)
        getAddress: (user: User | string) => Address | null;    // contravariance (function signature)
    }
``` 

Using the second option, the compiler will finally complain about the implementation:

```typescript
> Type 'string | User' is not assignable to type 'User'.
``` 

This is an easy way to avoid bugs that could be very hard to find, 
since TypeScript gives a very false sense of security in such cases.

**About the author: Michael Hammerl**

Michael has several years of experience as a full-stack developer. He specializes in TypeScript and Domain-driven design. Michael joined MaibornWolff in 2019.