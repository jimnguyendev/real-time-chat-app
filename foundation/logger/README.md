# Logger Package

The logger package provides a flexible and extensible logging system for Go applications with support for structured logging, trace IDs, and custom event handling.

## Features

- Multiple log levels (Debug, Info, Warn, Error)
- Structured JSON logging
- Trace ID support for distributed tracing
- Custom event handling
- Source file and line number tracking
- Standard library logger compatibility

## Usage

### Basic Setup

```go
logger := logger.New(
    os.Stdout,           // output writer
    logger.LevelInfo,    // minimum log level
    "my-service",       // service name
    func(ctx context.Context) string { // trace ID function
        return "trace-123"
    },
)
```

### Logging with Different Levels

```go
ctx := context.Background()

logger.Debug(ctx, "Debug message", "key", "value")
logger.Info(ctx, "Info message", "key", "value")
logger.Warn(ctx, "Warning message", "key", "value")
logger.Error(ctx, "Error message", "key", "value")
```

### Custom Call Stack Position

For each log level, there's a corresponding method with 'c' suffix that allows specifying the call stack position:

```go
logger.Debugc(ctx, 2, "Debug message", "key", "value")
logger.Infoc(ctx, 2, "Info message", "key", "value")
logger.Warnc(ctx, 2, "Warning message", "key", "value")
logger.Errorc(ctx, 2, "Error message", "key", "value")
```

### Event Handling

You can register custom event handlers for different log levels:

```go
events := logger.Events{
    Debug: func(ctx context.Context, r slog.Record) {
        // Custom debug event handling
    },
    Info: func(ctx context.Context, r slog.Record) {
        // Custom info event handling
    },
    // ... other event handlers
}

logger := logger.NewWithEvents(
    os.Stdout,
    logger.LevelInfo,
    "my-service",
    traceIDFn,
    events,
)
```

### Standard Library Logger

Create a standard library compatible logger:

```go
stdLogger := logger.NewStdLogger(logger, logger.LevelInfo)
```

## Output Format

The logger outputs structured JSON logs with the following fields:

- `time`: Timestamp of the log entry
- `level`: Log level (DEBUG, INFO, WARN, ERROR)
- `msg`: Log message
- `file`: Source file and line number
- `service`: Service name
- `trace_id`: Trace ID (if provided)
- Additional key-value pairs passed as arguments

Example output:
```json
{
    "time": "2024-03-13T10:30:00Z",
    "level": "INFO",
    "msg": "User logged in",
    "file": "auth.go:42",
    "service": "my-service",
    "trace_id": "trace-123",
    "user_id": "user-456"
}
```

## Best Practices

1. Always provide a context object for consistent tracing
2. Use structured logging with key-value pairs
3. Choose appropriate log levels based on message importance
4. Configure minimum log level based on environment
5. Implement trace ID function for distributed systems

## License

This package is part of the real-time-chat-app project.