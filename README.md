# Broadcast
Broadcast is a Go library that provides a mechanism for sending repeated notifications to multiple goroutines with guaranteed delivery and user-defined types.

# Why Broadcast?
### Why not Channels?
The standard way to handle notifications in Go is via a `chan struct{}`, but sending a message to a channel is received by a single goroutine. The only way to broadcast to multiple goroutines is to close the channel, which is not a viable option for sending repeated notifications.

Broadcast provides a solution for sending repeated notifications to multiple goroutines with guaranteed delivery.
### Why not sync.Cond?
`sync.Cond` is the standard solution based on condition variables to set up containers of goroutines waiting for a specific condition. However, the `Broadcast()` method doesn't guarantee that a goroutine will receive the notification. The notification will be lost if the listener goroutine isn't waiting on the `Wait()` method.

Broadcast provides a solution for sending repeated notifications to multiple goroutines with guaranteed delivery.

# How to Use Broadcast
### Step by Step
First, create a Relay for a message type (an empty struct in this example):
```go
relay := broadcast.NewRelay[struct{}]()
```
Once a Relay is created, create a new listener using the Listener method. As the Broadcast library relies internally on channels, it accepts a capacity:
```go
list := relay.Listener(1) // Create a new listener based on a channel with a one capacity
```
A Relay can send a notification in three different ways:
* `Notify`: block until a notification is sent to all the listeners
* `NotifyCtx`: send a notification to all listeners unless the provided context times out or is canceled
* `Broadcast`: send a notification to all listeners in a non-blocking manner; delivery isn't guaranteed

On the Listener side, we can access the internal channel using Ch:
```go
<-list.Ch() // Wait on a notification
```
We can close a Listener and a Relay using `Close`:
```go
list.Close() 
relay.Close()
```
Closing a Relay and Listeners can be done concurrently in a safe manner.

```go
type msg string

const (
    msgA msg = "A"
    msgB     = "B"
    msgC     = "C"
)

relay := broadcast.NewRelay[msg]() // Create a relay for msg values
defer relay.Close()

// Listener goroutines
for i := 0; i < 2; i++ {
    go func(i int) {
        l := relay.Listener(1)  // Create a listener with a buffer capacity of 1
        for n := range l.Ch() { // Ranges over notifications
            fmt.Printf("listener %d has received a notification: %v\n", i, n)
        }
    }(i)
}

// Notifiers
time.Sleep(time.Second)
relay.Notify(msgA)                                     // Send notification with guaranteed delivery
ctx, _ := context.WithTimeout(context.Background(), 0) // Context with immediate timeout
relay.NotifyCtx(ctx, msgB)                             // Send notification respecting context cancellation
time.Sleep(time.Second)                                // Allow time for previous messages to be processed
relay.Broadcast(msgC)                                  // Send notification without guaranteed delivery
time.Sleep(time.Second)                                // Allow time for previous messages to be processed
```
# Contributing
Contributing to this project is highly encouraged and appreciated. If you have ideas for new features or have spotted a bug, please open an issue on our GitHub repository. We will review the issue and work with you to find a solution.

If you want to contribute code to the project, feel free to propose a pull request. To ensure that your contribution is accepted, please follow these guidelines:
1. Fork the repository and create a new branch for your changes.
2. Make your changes and ensure that they are thoroughly tested.
3. Write clear and concise commit messages that explain your changes.
4. Ensure that your code follows the project's coding conventions and style.
5. Submit a pull request and describe your changes in detail.

We will review your pull request and provide feedback if necessary. Once your contribution is accepted, we will merge it into the project and credit you in the CONTRIBUTORS.md file.
If you have any questions or concerns about contributing to the project, please don't hesitate to contact us through GitHub or by reaching out to @supriyo. We are open to feedback and discussion and welcome any contributions to help improve this project.












