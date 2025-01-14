---
layout: post
title: "Handling Optional Parameters in Go with Functional Options"
date: 2025-01-14
tags: [go, functional, patterns]
---

Go is a powerful language that values simplicity and clarity, but newcomers might find its lack of support for optional parameters a limitation. For instance, if you're used to languages like Python or JavaScript, creating functions with default values for parameters might seem straightforward. However, with a few idiomatic techniques, Go provides robust alternatives for handling optional parameters.

In this post, we'll explore how to implement optional parameters in Go using the functional options pattern.

## The Problem: Optional Parameters in Go

Consider a scenario where you want to write a JSON response in an HTTP handler. A function for this might look like:

```go
func WriteJSON(w http.ResponseWriter, data any, status int, header http.Header) {
	w.Header().Set("Content-Type", "application/json")
	for key, values := range header {
		for _, value := range values {
			w.Header().Add(key, value)
		}
	}

	js, err := json.Marshal(data)
	if err != nil {
		http.Error(w, "failed to marshal JSON", http.StatusInternalServerError)
		return
	}

	w.WriteHeader(status)
	w.Write(js)
}
```

Here, the `status` and `header` parameters can be optional:
- If `status` is not set, it should default to `200`.
- If `header` is not set, it should be ignored.

While you can overload functions in other languages, Go doesn't support this. Fortunately, we can use the **functional options pattern** to address this.

## The Solution: Functional Options Pattern

### Step 1: Define an Options Struct

First, define a struct to hold optional parameters:

```go
type options struct {
	status *int
	header http.Header
}
```

Here:
- The `status` field is a pointer to an integer, allowing us to detect when it hasn’t been set.
- The `header` field is an `http.Header` to store response headers.

### Step 2: Define Option Type and Implementations

Next, define an `Option` type as a function that modifies the `options` struct:

```go
type Option func(*options) error

func WithStatus(status int) Option {
	return func(o *options) error {
		if status < 100 || status > 599 {
			return fmt.Errorf("invalid status code: %d", status)
		}
		o.status = &status
		return nil
	}
}

func WithHeader(header http.Header) Option {
	return func(o *options) error {
		o.header = header
		return nil
	}
}
```

These functions allow you to specify the status code or headers in a clean, modular way.

### Step 3: Update the WriteJSON Function

Modify `WriteJSON` to accept a variadic slice of `Option` arguments:

```go
func WriteJSON(w http.ResponseWriter, data any, opts ...Option) {
	// Evaluate the options
	var options options
	for _, opt := range opts {
		if err := opt(&options); err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}
	}

	// Set the status code to 200 OK if not specified
	status := http.StatusOK
	if options.status != nil {
		status = *options.status
	}

	// Set the response headers
	w.Header().Set("Content-Type", "application/json")
	for k, v := range options.header {
		w.Header()[k] = v
	}

	// Marshal the data to JSON
	js, err := json.Marshal(data)
	if err != nil {
		http.Error(w, "failed to marshal JSON", http.StatusInternalServerError)
		return
	}

	w.WriteHeader(status)
	w.Write(js)
}
```

### Step 4: Use the Function

Here’s how you can use the updated `WriteJSON` function:

```go
func handler(w http.ResponseWriter, r *http.Request) {
	data := map[string]string{"message": "Hello, world!"}

	// Default status and no headers
	WriteJSON(w, data)

	// Custom status
	WriteJSON(w, data, WithStatus(http.StatusCreated))

	// Custom status and headers
	headers := http.Header{"X-Custom-Header": []string{"value"}}
	WriteJSON(w, data, WithStatus(http.StatusAccepted), WithHeader(headers))
}
```

### Advantages of Functional Options

1. **Flexibility**: Easily add new optional parameters without breaking existing function signatures.
2. **Readability**: Clear and descriptive usage with named option constructors.
3. **Error Handling**: Validate inputs when constructing options.

### Considerations

- **Complexity**: The pattern introduces additional code for simple cases.
- **Overhead**: Parsing options adds minor runtime overhead.

## Conclusion

While Go doesn’t support optional parameters natively, the functional options pattern provides an idiomatic and flexible solution. It enables you to write clean, extensible functions that handle optional parameters gracefully.

Give it a try in your next Go project and enjoy the power of functional options!
