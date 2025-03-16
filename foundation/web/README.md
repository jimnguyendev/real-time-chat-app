# Web Package

## Overview

The `web` package provides a lightweight web framework extension for Go applications. It's designed to simplify the creation of HTTP services with features like middleware support, CORS handling, request/response processing, and static file serving capabilities.

This package is built on top of Go's standard `net/http` package and integrates with OpenTelemetry for distributed tracing.

## Key Components

### App

The `App` struct is the central component of the framework. It serves as the entry point for HTTP requests and manages the application's routing, middleware, and configuration.

```go
type App struct {
    log     Logger
    tracer  trace.Tracer
    mux     *http.ServeMux
    otmux   http.Handler
    mw      []MidFunc
    origins []string
}
```

#### Creating a New App

```go
func NewApp(log Logger, tracer trace.Tracer, mw ...MidFunc) *App
```

The `NewApp` function creates a new `App` instance with the provided logger, tracer, and middleware functions.

### Handler Functions

The package defines a custom handler function type that works within the framework:

```go
type HandlerFunc func(ctx context.Context, r *http.Request) Encoder
```

Unlike standard HTTP handlers, these functions receive a context and return an `Encoder` that defines how to encode the response.

### Registering Routes

The `App` struct provides several methods for registering routes:

- `HandlerFunc`: Registers a handler function with middleware support and OpenTelemetry tracing.
- `HandlerFuncNoMid`: Registers a handler function without middleware or tracing.
- `RawHandlerFunc`: Registers a standard `http.HandlerFunc` with middleware support.

### Middleware

The package includes a middleware system that allows for pre and post-processing of requests:

```go
type MidFunc func(handler HandlerFunc) HandlerFunc
```

Middleware functions can be applied globally to all routes or to specific routes.

### CORS Support

The package provides built-in CORS support through the `EnableCORS` method:

```go
func (a *App) EnableCORS(origins []string)
```

This method configures the application to handle CORS preflight requests and set appropriate headers.

### Static File Serving

The package includes two methods for serving static files:

- `FileServer`: Serves static files from an embedded filesystem.
- `FileServerReact`: Serves a React single-page application with proper routing support.

### Request Processing

The package provides utilities for processing HTTP requests:

- `Param`: Extracts path parameters from the request.
- `Decode`: Decodes the request body into a data model.

### Response Processing

The package includes utilities for sending HTTP responses:

- `Respond`: Sends a response to the client with proper content type and status code.
- `Encoder` interface: Defines how to encode a response.

## Usage Examples

### Basic Setup

```go
package main

import (
    "context"
    "log"
    "net/http"
    
    "path/to/foundation/web"
    "go.opentelemetry.io/otel/trace"
)

func main() {
    // Create a logger function
    logger := func(ctx context.Context, msg string, args ...any) {
        log.Printf(msg, args...)
    }
    
    // Create a tracer
    tracer := trace.NewNoopTracerProvider().Tracer("example")
    
    // Create a new app
    app := web.NewApp(logger, tracer)
    
    // Register routes
    app.HandlerFunc(http.MethodGet, "api", "/hello", helloHandler)
    
    // Start the server
    if err := http.ListenAndServe(":8080", app); err != nil {
        log.Fatal("server error:", err)
    }
}

func helloHandler(ctx context.Context, r *http.Request) web.Encoder {
    // Return a response
    return &response{message: "Hello, World!"}
}

type response struct {
    message string
}

func (r *response) Encode() ([]byte, string, error) {
    return []byte(r.message), "text/plain", nil
}
```

### Using Middleware

```go
// Define a middleware function
func loggingMiddleware(handler web.HandlerFunc) web.HandlerFunc {
    return func(ctx context.Context, r *http.Request) web.Encoder {
        log.Printf("Request: %s %s", r.Method, r.URL.Path)
        return handler(ctx, r)
    }
}

// Apply middleware to all routes
app := web.NewApp(logger, tracer, loggingMiddleware)

// Or apply middleware to a specific route
app.HandlerFunc(http.MethodGet, "api", "/hello", helloHandler, loggingMiddleware)
```

### Enabling CORS

```go
// Enable CORS for all origins
app.EnableCORS([]string{"*"})

// Enable CORS for specific origins
app.EnableCORS([]string{"https://example.com", "https://api.example.com"})
```

### Serving Static Files

```go
//go:embed static
var staticFiles embed.FS

// Serve static files
if err := app.FileServer(staticFiles, "static", "/static/"); err != nil {
    log.Fatal("failed to set up file server:", err)
}

// Serve a React application
if err := app.FileServerReact(staticFiles, "static", "/"); err != nil {
    log.Fatal("failed to set up React file server:", err)
}
```

## Implementation Details

### Context Management

The package uses context values to store and retrieve the HTTP response writer and tracer. This allows handlers to access these values without passing them explicitly.

### OpenTelemetry Integration

The package integrates with OpenTelemetry to provide distributed tracing. Each request is wrapped in a span, and the package injects trace context into response headers.

### Error Handling

The package includes utilities for handling errors in HTTP responses. If a handler returns an error, it will be converted to an appropriate HTTP status code.

## Best Practices

1. **Use Middleware for Cross-Cutting Concerns**: Authentication, logging, and error handling are good candidates for middleware.

2. **Implement the Encoder Interface**: Create custom response types that implement the `Encoder` interface for consistent response handling.

3. **Use Context for Request-Scoped Values**: Store request-scoped values in the context for access throughout the request lifecycle.

4. **Enable CORS When Needed**: Only enable CORS when your API needs to be accessed from different origins.

5. **Use Path Groups for API Versioning**: The group parameter in handler registration methods can be used for API versioning (e.g., "v1", "v2").