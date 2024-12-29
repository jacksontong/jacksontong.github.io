---
layout: post
title: "Simple Inversion of Control Container in Go"
date: 2024-12-29
tags: go ioc
---

Inversion of Control (IoC) containers are a design pattern used to manage dependencies in a software application. They help decouple the creation of objects from their usage, making the code more modular and easier to test.

There are many libraries in Go to help implement dependency injection, such as [Dig](https://github.com/uber-go/dig), [Wire](https://github.com/google/wire), and [Fx](https://github.com/uber-go/fx). However, they are not always easy to use and often require a significant learning curve.

This post demonstrates a simple IoC container in Go that addresses these challenges by providing a thread-safe way to register and resolve services.

Here is how to implement a simple IoC container:

### Step 1: Define the Container Type and Containers Variable

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

var containers = &container{
	services: make(map[string]interface{}),
}
```

The `container` struct holds the registered services in a map and provides thread-safe access using a read-write mutex. The `containers` variable is an instance of the `container` struct.

### Step 2: Define the Register Function

```go
func Register(name string, service interface{}) {
	containers.mu.Lock()
	defer containers.mu.Unlock()
	containers.services[name] = service
}
```

The `Register` function adds a service to the container with the given name. It locks the container for writing to ensure thread safety.

### Step 3: Define the Resolve Function

```go
func Resolve[T any](name string) (T, error) {
	containers.mu.RLock()
	defer containers.mu.RUnlock()

	service, exists := containers.services[name]
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

The `Resolve` function retrieves a service from the container by name and casts it to the specified type. It locks the container for reading to ensure thread safety.

### Usage

To use this IoC container, you can register and resolve services as shown below:

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

This simple IoC container can be extended to support more advanced features as needed. I hope you find this example useful for your Go projects.

Inversion of Control is a powerful technique, but it's easy to make a mistake. You should read this post from Martin Fowler to have a better understanding: [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html).

You can find the full source code at:
1. [https://gist.github.com/jacksontong/b05f00458416cb830ad6820a9cfc59f4](https://gist.github.com/jacksontong/b05f00458416cb830ad6820a9cfc59f4).
2. https://go.dev/play/p/6Aw5hTNXXBQ
