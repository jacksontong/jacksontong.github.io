---
layout: post
title: "Setting Up a VSCode Devcontainer Using Docker Compose"
date: 2024-12-31
tags: go docker postgres
---

Setting up a development environment can sometimes be challenging, especially when multiple services are involved. Fortunately, with Docker Compose and VSCode Devcontainers, you can create a streamlined setup that includes all necessary dependencies and services for your project.

This tutorial will guide you through setting up a VSCode Devcontainer for a Go project using Docker Compose. With this setup, you can run multiple services like PostgreSQL and pgAdmin alongside your application.

## Step 1: Create a `.env` File

First, create a `.env` file to store your database credentials and other environment-specific configurations:

```env
DB_USER=dev
DB_PASSWORD=secret
DB_NAME=dev_db
DB_HOST=db

PGADMIN_DEFAULT_EMAIL=anonymous@example.com
PGADMIN_DEFAULT_PASSWORD=secret
```

This file centralizes environment variables, making them easier to manage and preventing hardcoding sensitive information into your codebase.

## Step 2: Create a `docker-compose.yaml` File

Next, define your services in a `docker-compose.yaml` file. Here is an example configuration:

```yaml
services:
  app:
    image: mcr.microsoft.com/vscode/devcontainers/go:1.23-bookworm
    ports:
      - "${APP_PORT:-8080}:8080"
    volumes:
      - .:/workspace:cached
    command: sleep infinity
  db:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "${PGADMIN_PORT:-5050}:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD}

volumes:
  db-data:
```

### Explanation:
- **app**: The Go development container. You can use any Go image here; this example uses the Microsoft-based image because it comes pre-packaged with tools and configurations that work well in a Devcontainer environment. The `command: sleep infinity` ensures the container stays alive while developing.
- **db**: A PostgreSQL service with a mounted volume for persistent storage.
- **pgadmin**: A pgAdmin service for managing your PostgreSQL database via a web interface.
- **volumes**: Shared volume for the database to persist data.

## Step 3: Create a `devcontainer.json` File

Configure the Devcontainer settings by creating a `.devcontainer/devcontainer.json` file:

```json
{
  "name": "Go Development Container",
  "dockerComposeFile": "../docker-compose.yaml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "customizations": {
    "vscode": {
      "extensions": ["golang.Go", "EditorConfig.EditorConfig"],
      "settings": {
        "terminal.integrated.defaultProfile.linux": "zsh",
        "terminal.integrated.profiles.linux": { "zsh": { "path": "/bin/zsh" } }
      }
    }
  },
  "remoteUser": "vscode",
  "shutdownAction": "stopCompose"
}
```

### Key Configuration:
- **dockerComposeFile**: Links to your `docker-compose.yaml` file.
- **service**: Specifies the primary container (the Go development container).
- **workspaceFolder**: Sets the folder inside the container to map your project files.
- **customizations**: Installs VSCode extensions for Go development and configures the terminal to use Zsh.
- **remoteUser**: Ensures all operations inside the container run as the `vscode` user.

## Step 4: Create a Go Application to Test Database Connection

To ensure your setup works, create a `main.go` file that tests the connection to the PostgreSQL database:

```go
package main

import (
	"database/sql"
	"fmt"
	"log"
	"os"

	_ "github.com/joho/godotenv/autoload"
	_ "github.com/lib/pq"
)

func main() {
	dbUser := os.Getenv("DB_USER")
	dbName := os.Getenv("DB_NAME")
	dbPass := os.Getenv("DB_PASSWORD")
	dbHost := os.Getenv("DB_HOST")

	connStr := fmt.Sprintf(
		"host=%s user=%s password=%s dbname=%s sslmode=disable",
		dbHost, dbUser, dbPass, dbName,
	)

	// Open a connection to the database
	db, err := sql.Open("postgres", connStr)
	if err != nil {
		log.Fatalf("Failed to open database: %v", err)
	}
	defer db.Close()

	// Ping the database to verify the connection
	if err := db.Ping(); err != nil {
		log.Fatalf("Failed to ping database: %v", err)
	}

	fmt.Println("Successfully connected to the database!")
}
```

### Key Points:
- This code uses environment variables to configure the database connection.
- It connects to the PostgreSQL database and verifies the connection using `db.Ping()`.

## Step 5: Open Project in VSCode Devcontainer

Open the project in VSCode Devcontainer. Once the container is running, execute the following command to test the database connection:

```bash
go run main.go
```

You should see the output:

```
Successfully connected to the database!
```

## Accessing pgAdmin

After starting the containers, you can access pgAdmin at [http://localhost:5050](http://localhost:5050). Use the credentials you specified in the environment variables to log in.

## Source Code and Documentation

- **Source Code**: The complete setup can be found on [GitHub](https://github.com/jacksontong/devcontainer-compose).
- **VSCode Devcontainer Documentation**: For more information on configuring Devcontainers, visit the [official VSCode documentation](https://code.visualstudio.com/docs/devcontainers/containers).

## Conclusion

Using Docker Compose with VSCode Devcontainers provides a powerful and efficient way to manage a development environment with multiple services. This tutorial showed how to set up a Go project with PostgreSQL and pgAdmin, enabling seamless integration and testing. Feel free to expand this setup with additional services as needed for your projects!
