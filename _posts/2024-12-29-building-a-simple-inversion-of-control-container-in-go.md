---
layout: post
title: "Building a Simple Inversion of Control Container in Go"
date: 2024-12-29
tags: go ioc
---

Inversion of Control (IoC) is a design principle that plays a key role in modern software architecture. IoC containers help manage dependencies by decoupling object creation from object usage, resulting in cleaner, more modular, and testable code.

While Go provides excellent support for dependency injection through libraries like [Dig](https://github.com/uber-go/dig), [Wire](https://github.com/google/wire), and [Fx](https://github.com/uber-go/fx), these libraries can be complex and require a steep learning curve. For simpler use cases, a lightweight IoC container might be a better fit.

This post demonstrates how to create a straightforward IoC container in Go. The implementation is designed to be thread-safe, easy to use, and minimalistic.

## Implementing the IoC Container

### Step 1: Define the Container Type and Container Variable

```go
package ioc

import (
	"fmt"
	"sync"
)

type container struct {
	services map[string]interface{}
	mu       sync.RWMutex
}

var globalContainer = &container{
	services: make(map[string]interface{}),
}
```

The `container` struct stores registered services in a map and ensures thread-safe access using a read-write mutex. The `globalContainer` variable is a singleton instance of the `container` struct.

### Step 2: Define the Register Function

```go
func Register(name string, service interface{}) {
	globalContainer.mu.Lock()
	defer globalContainer.mu.Unlock()
	globalContainer.services[name] = service
}
```

The `Register` function associates a name with a service and locks the container for writing to ensure thread safety.

### Step 3: Define the Resolve Function

```go
func Resolve[T any](name string) (T, error) {
	globalContainer.mu.RLock()
	defer globalContainer.mu.RUnlock()

	service, exists := globalContainer.services[name]
	if !exists {
		var zero T
		return zero, fmt.Errorf("service not found: %s", name)
	}

	result, ok := service.(T)
	if !ok {
		var zero T
		return zero, fmt.Errorf("service type mismatch: expected %T", zero)
	}

	return result, nil
}
```

The `Resolve` function retrieves a registered service by name and casts it to the specified type. It locks the container for reading, ensuring that concurrent reads are safe.

## Using the IoC Container

Here is an example of how to use the IoC container to register and resolve a service:

```go
package main

import (
	"fmt"
	"ioc"
)

const myServiceName = "myService"

type MyService struct {
	Name string
}

func main() {
	// Register a service
	ioc.Register(myServiceName, &MyService{Name: "Test Service"})

	// Resolve the service
	service, err := ioc.Resolve[*MyService](myServiceName)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}

	fmt.Println("Service Name:", service.Name)
}
```

This example registers a service with the container and then resolves it by name, demonstrating both the flexibility and simplicity of the implementation.

## Enhancements and Extensions

This basic IoC container can serve as a foundation for more advanced features, such as:

- **Lifecycle Management**: Adding support for transient, singleton, or scoped lifecycles.
- **Dependency Graph Resolution**: Automatically resolving dependencies for complex services.
- **Configuration Support**: Integrating with configuration files or environment variables for dynamic service registration.

## Conclusion

Building a simple IoC container in Go is a great exercise to understand dependency injection and its benefits. While lightweight implementations like this are sufficient for many use cases, consider established libraries for larger or more complex projects.

For a deeper understanding of IoC and dependency injection, check out [Martin Fowler's article](https://martinfowler.com/articles/injection.html).

You can explore the full source code here:

1. [Gist: Simple IoC Container](https://gist.github.com/jacksontong/b05f00458416cb830ad6820a9cfc59f4)
2. [Go Playground Example](https://go.dev/play/p/jgDjMHZNQBf)

I hope this guide helps you in your Go projects. If you have questions or suggestions, feel free to share them in the comments!
