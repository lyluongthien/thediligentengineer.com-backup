---
title: "The C in Golang ? Let's glance !"
datePublished: Mon Dec 11 2023 13:34:13 GMT+0000 (Coordinated Universal Time)
cuid: clq0yga2k000809jsclrfh7e1
slug: the-c-in-golang-lets-glance
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702301025753/f0671959-5929-485b-b063-7e9139a4c422.jpeg
tags: c, golang, go-cjffccfnf0024tjs1mcwab09t

---


Go (Golang) is renowned for its simplicity, performance, and ease of use. However, there are instances where interacting with C libraries becomes necessary for various reasons, such as leveraging existing codebases or accessing hardware-specific functionality. The CGO (C Go) tool in Go facilitates this interaction, allowing developers to seamlessly integrate Go code with C libraries. In this article, we will explore the fundamentals of CGO and provide real-world examples to illustrate its application.

## Understanding CGO:
CGO acts as a bridge between Go and C, enabling Go code to call C functions and vice versa. It is particularly useful when interfacing with low-level libraries, system calls, or when performance optimization is critical.

## Setting Up a Simple CGO Project:
Let's start with a basic example of calling a C function from Go. Consider the following C code in a file named example.c:

```c
// example.c
#include <stdio.h>

void helloCGO() {
    printf("Hello from CGO!\n");
}```
Now, create a Go file, let's call it main.go, to call the C function:

```go
// main.go
package main

/*
#cgo LDFLAGS: -lexample
#include "example.c"
*/
import "C"

func main() {
    C.helloCGO()
}```
In this example, the #cgo directive is used to link the C library with the Go code. The helloCGO function from the C code is then called from the Go main function.

## Passing Data Between Go and C:
One of the powerful features of CGO is the ability to pass data between Go and C seamlessly. Consider the following example where we pass a string from Go to a C function and receive the result back:

```go
// main.go
package main

/*
#cgo LDFLAGS: -lexample
#include "example.c"
*/
import "C"
import "fmt"

func main() {
    goStr := "Hello from Go!"
    cStr := C.CString(goStr)
    defer C.free(unsafe.Pointer(cStr))

    result := C.processString(cStr)
    fmt.Printf("Result from C: %s\n", C.GoString(result))
}```
In this example, the CString function is used to convert a Go string to a C string, and GoString is used to convert a C string back to a Go string. The processString C function receives the C string, performs some processing, and returns the result.

## Integrating with External C Libraries:
CGO also allows seamless integration with external C libraries. Suppose we want to use the popular SQLite database in our Go application. We can leverage CGO to interface with SQLite:

```go
// main.go
package main

/*
#cgo LDFLAGS: -lsqlite3
#include <sqlite3.h>
*/
import "C"
import "fmt"

func main() {
    var db *C.sqlite3
    if rc := C.sqlite3_open(C.CString("example.db"), &db); rc == C.SQLITE_OK {
        defer C.sqlite3_close(db)
        fmt.Println("Connected to SQLite database!")
    } else {
        fmt.Println("Failed to open SQLite database.")
    }
}```
In this example, the SQLite library is linked using the #cgo directive, and Go code interfaces with SQLite through the C functions provided by the library.

## Performance Optimization with CGO:
CGO can also be employed for performance optimization by leveraging C libraries for computationally intensive tasks. For example, integrating the GNU Scientific Library (GSL) for advanced mathematical operations:

```go
// main.go
package main

/*
#cgo LDFLAGS: -lgsl -lgslcblas
#include <gsl/gsl_math.h>
*/
import "C"
import "fmt"

func main() {
    result := C.gsl_sf_bessel_J0(C.double(5.0))
    fmt.Printf("Result of Bessel function: %f\n", float64(result))
}```
Here, we use the Bessel function from GSL to demonstrate how CGO can be utilized for numerical computations.

## Conclusion:
CGO opens up a world of possibilities for Go developers, enabling seamless integration with C libraries and facilitating tasks that require low-level system interaction or performance optimization. The real-world examples provided here showcase the versatility and power of CGO in various scenarios. As you embark on incorporating CGO into your projects, keep in mind its potential for enhancing performance, leveraging existing C codebases, and accessing hardware-specific functionality, making it a valuable tool in the Go developer's toolkit.