---
title: "Go: When Does Stack Turn Into Heap?"
datePublished: 2026-02-24T17:18:41.950Z
cuid: cmm0vdjx6000b1qm56wqu2f97
slug: go-when-does-stack-turn-into-heap
cover: https://cdn.hashnode.com/uploads/covers/60391207ee9954619ef4df72/08c33eab-ea4d-4fed-b7e9-f1f875d5e04f.png
ogImage: https://cdn.hashnode.com/uploads/og-images/60391207ee9954619ef4df72/d0e88de1-2952-4e4b-b9a0-f5b7e67b8970.jpg
tags: cpp, c, go, performance, rust, compiler, memory-management, compilers, escape-analysis

---

* * *

In Go, you don’t choose stack vs heap.

The compiler does.

Most local variables live on the stack - fast and cheap.  
But sometimes a value gets promoted to the heap instead.

Why?

Because of [***escape analysis***](https://go.dev/src/cmd/compile/internal/escape/escape.go).

* * *

## The Classic Example

```go
func foo() *int {
    x := 10
    return &x
}
```

At first glance, this looks dangerous.  
You’re returning a pointer to a local variable.

In C, this is undefined behavior.  
In Go?

Totally fine.

The compiler sees that `x` escapes the function scope, so it allocates `x` on the heap.

Conceptually, it becomes:

```go
x := new(int)
*x = 10
return x
```

You didn’t explicitly ask for heap allocation.  
But you got one.

* * *

## What Does “Escape” Actually Mean?

A variable escapes when it might outlive the function stack frame.

More precisely: if the compiler cannot prove that a value stays within the current frame, it assumes the value escapes.

Common triggers:

### 1️⃣ Returning a pointer

```go
return &x
```

If the value may outlive the function, it escapes.

Note: returning a pointer does not *always* force heap allocation.  
If the compiler can prove, often through inlining, that the value does not actually outlive the caller, it may still keep it on the stack.

Escape analysis is conservative, not naive.

* * *

### 2️⃣ Captured by a closure

```go
func foo() func() int {
    x := 10
    return func() int { return x }
}
```

The closure may live longer than `foo()`.  
If the compiler cannot prove otherwise, `x` goes to the heap.

* * *

### 3️⃣ Used in a goroutine

```go
go func() {
    println(x)
}()
```

Now execution is asynchronous.  
The compiler cannot guarantee that `x` dies before the goroutine uses it.

Conservatively, heap allocation.

* * *

### 4️⃣ Stored inside something that escapes

If a struct, slice, or interface value escapes, anything it references may escape too.

Escape is transitive.

* * *

## Important: Escape Analysis Is Conservative

Go’s compiler must be correct.

If it is unsure whether something escapes, it assumes it does.

It would rather allocate on the heap than risk memory unsafety.

That means:

*   Escape analysis never makes your program incorrect
    
*   But it *can* make it slower
    

* * *

## Why This Is Subtle (Rust Helps Explain ;) )

In Rust, the equivalent code does not compile:

```rust
fn foo() -> &i32 {
    let x = 10;
    &x
}
```

Rust forces you to model ownership explicitly.

Go does not.

Instead of rejecting your program, Go changes the allocation strategy behind the scenes.

In Rust, memory rules are visible in the type system.  
In Go, they live inside the compiler.

You do not see the decision.

Unless you run:

```shell
go build -gcflags="-m"
```

And then the compiler prints:

```plaintext
moved to heap: x
```

* * *

## Because Memory Is Serious Enough, so:

Go:

> “I moved it to the heap for safety.”

You:

> “But I didn’t ask for that.”

Go:

> “You’re welcome.”

🙂

* * *

## Why You Should Care

Heap allocations mean:

*   More garbage collection
    
*   More latency variance
    
*   More pressure under load
    

For most applications, it does not matter.

For high performance systems, it absolutely does.

Understanding when stack turns into heap helps you:

*   Reduce allocations
    
*   Control GC pressure
    
*   Write more predictable code
    
*   Interpret `-gcflags="-m"` output with confidence
    

* * *

## So?

Go gives you simplicity.

That simplicity is powered by a sophisticated compiler performing escape analysis behind the scenes.

Stack vs heap in Go is not a syntax decision.

It is a compiler decision.

And sometimes it decides differently than you expect.

Understanding that boundary is one of the steps from writing Go that works to writing Go that performs.

\_

\_

\_

\_

* * *

## P.S. Why Is This (returing var addr) Undefined Behavior in C? 😉

*glad you scroll to this hidden one ;)))*

In C, this code is undefined behavior:

```c
int* foo() {
    int x = 10;
    return &x;  // nah, dangling pointer
}
```

Why?

Because `x` lives on the stack, and its lifetime ends when `foo()` returns.  
Returning `&x` creates a **dangling pointer** — a pointer to memory that is no longer valid.

After the function exits:

*   The stack frame is popped
    
*   That memory may be reused
    
*   The pointer now points to garbage
    

The C standard does not define what happens if you dereference it. It might:

*   Seem to work
    
*   Return random data
    
*   Crash
    
*   Corrupt memory
    
*   Break only in release builds
    

All of those are legal outcomes.

“Undefined behavior” means the compiler assumes you never do this.  
So it’s free to optimize aggressively under that assumption.

That’s why:

*   Go silently moves the value to the heap
    
*   Rust refuses to compile
    
*   C says: “You promised you knew what you were doing.”
    

And that promise is where things usually go wrong.