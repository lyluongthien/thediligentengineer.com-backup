---
title: "A note on TypeScript discrimination unions"
datePublished: Thu Jan 11 2024 04:26:15 GMT+0000 (Coordinated Universal Time)
cuid: clr8pj0ao000d0ajxcp2m67km
slug: a-note-on-typescript-discrimination-unions
tags: typescript

---

Recently I started using discrimination unions a lot in typescript.
This technique allows us to write more type-safe codes, for instance:
```typescript
interface Bird {
  name: "bird",
  fly(): void;
  layEggs(): void;
}
 
interface Fish {
  name: "fish",
  swim(): void;
  layEggs(): void;
}
 
declare function getSmallPet(): Fish | Bird;
 
let pet = getSmallPet();

```
This works:
```typescript
pet.layEggs();
```
But not this:
```typescript 
pet.swim(); 
```
```diff
- >> Property 'swim' does not exist on type 'Bird | Fish'.
- Property 'swim' does not exist on type 'Bird'.
```
This forces us to add additional checks to pass the compiler error:
```typescript
if(pet.name === 'fish'){
  pet.swim(); // this will work
}
```
 See more on [typescriptlang.org/docs](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#discriminating-unions) 
