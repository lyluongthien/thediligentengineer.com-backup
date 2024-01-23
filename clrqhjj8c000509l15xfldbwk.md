---
title: "Introduction to the new thread-safe event package for Golang"
datePublished: Tue Jan 23 2024 15:02:34 GMT+0000 (Coordinated Universal Time)
cuid: clrqhjj8c000509l15xfldbwk
slug: introduction-to-the-new-thread-safe-event-package-for-golang
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1706022099050/151b1551-d2d7-48f5-a9ba-fa2ee54a2436.png
tags: go, package

---


 
Events are a common pattern in many applications, especially those that involve user interactions, asynchronous operations, or communication between different components. Events allow you to decouple the logic of the event source from the logic of the event listeners, making your code more modular, reusable, and testable.

However, managing events and listeners in Go can be tricky, especially when concurrency is involved. You need to ensure that your event channels are properly created, closed, and buffered, and that your listeners are registered and unregistered correctly. You also need to handle any errors or panics that may occur during the event processing.

That's why we created the [`event`](https://pkg.go.dev/github.com/lltpkg/event) package, a simple and thread-safe mechanism for managing events and listeners in Go applications. The event package provides a high-level API that abstracts away the low-level details of creating and managing event channels and listeners. It also handles any errors or panics gracefully, ensuring that your application does not crash or leak resources.

## Installation
To use the event package in your Go project, you can use the following go get command:
```bash
go get -u github.com/lltpkg/event
```
## Quick start
To get started with the event package, you need to import it in your Go file:
```go
import "github.com/lltpkg/event"
```
Then, you can create an event channel and a listener for any event name you want. For example, let's create an event channel and a listener for the "Greeting" event:
```go
package main

import (
"fmt"

"github.com/lltpkg/event"
)

func main() {
// Create an event channel and a cleanup function for the "Greeting" event
evChan, cleanup := event.EventChannel("Greeting")
// Make sure to call the cleanup function when done
defer cleanup()

// Listen for the "Greeting" event in a goroutine
go func() {
// Receive the event data from the channel
receivedData := <-evChan
// Print the greeting message
fmt.Println("Hello,", receivedData)
}()

// Trigger the "Greeting" event with some data
event.FireEvent("Greeting", "World")
}
```
If you run this code, you should see the following output:
``` txt
Hello, World
```
As you can see, the event package makes it easy to create and use event channels and listeners in Go. You don't need to worry about creating, closing, or buffering the channels, or registering or unregistering the listeners. The event package takes care of all that for you.

##  Usage
Creating Events and Listeners
The event package allows you to create named events and associate listeners with them. You can use the EventChannel function to create an event channel and a cleanup function for any event name:
```go
// Create an event channel and a cleanup function for "exampleEvent"
eventChan, cleanup := event.EventChannel("exampleEvent")
// Make sure to call the cleanup function when done
defer cleanup()
```
The event channel is a chan interface{} that receives the event data whenever the event is triggered. The cleanup function is a func() that closes the event channel and unregisters the listener from the event. You should always call the cleanup function when you are done with the event channel, otherwise you may leak resources or cause deadlocks.

You can create as many event channels and listeners as you want for the same or different event names. The event package will ensure that each listener receives the event data in a thread-safe manner.

## Triggering Events
You can trigger events using the FireEvent function. This function allows you to send data to all registered listeners for a specific event name:
```go
// Trigger the "exampleEvent" with some data
event.FireEvent("exampleEvent", "event data")
```
The data can be any value that implements the `interface{}` type. The `FireEvent` function will send the data to all the event channels that are listening for the event name. The function will also handle any errors or panics that may occur during the event processing, and log them using the standard log package.

You can trigger events from anywhere in your code, as long as you import the event package. The `FireEvent` function is thread-safe and non-blocking, so you can use it in concurrent or asynchronous contexts.


We hope that you find the event package useful and easy to use. If you have any feedback, suggestions, or issues, please feel free to open an issue or a pull request on the [GitHub repository](github.com/lltpkg/event). We appreciate any contributions that make the event package more robust and versatile.
 