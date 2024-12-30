---
layout: post
title: "A Simple Approach to Managing Configuration in Go"
date: 2024-12-30
tags: go config
---

Managing configuration in a Go project can be simple and highly productive, enhancing both maintainability and development speed. For small to medium-sized projects, a straightforward solution can often be more effective and easier to maintain. In this post, we’ll explore a lightweight and custom approach to handling configuration in Go using struct tags and environment variables.

## The Simple Approach: Using Struct Tags and Reflection

Here is an example implementation for managing configuration:

### Step 1: Define the Config Struct

```go
type Config struct {
	AppPort       string `mapstructure:"APP_PORT"`
	Environment   string `mapstructure:"ENVIRONMENT"`
}
```

The `Config` struct serves as the blueprint for your application's configuration. Each field represents a configuration value, annotated with a `mapstructure` tag to specify the corresponding environment variable name.

- **Struct Tags**: The `mapstructure` tag explicitly defines which environment variable maps to each field. For example, `APP_PORT` maps to `AppPort`.
- **Fallback Mechanism**: If no `mapstructure` tag is provided, the field name (converted to uppercase) is used as the default environment variable name.

### Step 2: Define the LoadConfig Function

```go
package config

import (
    "os"
    "reflect"
    "strings"
)

func LoadConfig() (*Config, error) {
    cfg := &Config{
        AppPort:     "8080",
        Environment: "development",
    }

    v := reflect.ValueOf(cfg).Elem()
    for i := 0; i < v.NumField(); i++ {
        field := v.Type().Field(i)
        envVar := field.Tag.Get("mapstructure")
        if envVar == "" {
            envVar = strings.ToUpper(field.Name)
        }
        if value, exists := os.LookupEnv(envVar); exists {
            v.Field(i).SetString(value)
        }
    }

    return cfg, nil
}
```

The `LoadConfig` function populates the `Config` struct with values from environment variables:

- **Default Values**: Initializes the struct with sensible defaults (e.g., `AppPort` is set to `8080` and `Environment` to `development`).
- **Reflection**: Iterates over each struct field using Go’s `reflect` package.
- **Environment Variable Lookup**: Checks for environment variables matching the `mapstructure` tag or field name and assigns their values to the corresponding struct fields.

### Example Usage

For local development, you can use this package in conjunction with [godotenv](https://github.com/joho/godotenv) to load environment variables from a `.env` file. This approach allows you to manage environment variables in a single file without cluttering your terminal or deployment configuration.

Create a `.env` file for local development:

```env
APP_PORT=3000
ENVIRONMENT=local
```

```go
package main

import (
    "fmt"
    "log"

    // auto load environment variables from .env file.
    _ "github.com/joho/godotenv/autoload"
    "your_project/config"
)

func main() {
    cfg, err := config.LoadConfig()
    if err != nil {
        log.Fatalf("Failed to load configuration: %v", err)
    }

    fmt.Printf("App is running on port: %s in %s environment\n", cfg.AppPort, cfg.Environment)
}
```

### Advantages of This Approach

- **Simplicity**: No external dependencies or libraries are required.
- **Environment-Driven**: Reads directly from environment variables, which is ideal for containerized and cloud-based applications.
- **Customizable**: Supports flexible mapping using struct tags or defaulting to field names.

### Setting Environment Variables

To run your application with this configuration loader, set the required environment variables. For example:

```bash
export APP_PORT=3000
export ENVIRONMENT=production
```

### Limitations

While this approach is simple and effective, it does have some limitations:

- **Basic Type Support**: This implementation works well for string values but would need extension for other types like integers or booleans.
- **No Validation**: It assumes that the environment variables are correctly set without validating them.

### Extending the Solution

To address these limitations, you could:

- Add support for type conversion for fields requiring non-string values.
- Include validation logic to ensure required fields are populated correctly.

## Conclusion

This lightweight configuration loader is a great fit for projects where simplicity and minimalism are key. By leveraging Go’s reflection capabilities, you can create a flexible and environment-driven configuration system without relying on external libraries. For larger projects, consider extending this solution or exploring established libraries if advanced features are needed.

You can try out the code on the Go Playground: [https://go.dev/play/p/EhjsUVOCGPO](https://go.dev/play/p/EhjsUVOCGPO).
