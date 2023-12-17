---
title: "Another article about Functional Options Pattern in Golang"
datePublished: Mon Dec 11 2023 13:19:41 GMT+0000 (Coordinated Universal Time)
cuid: clq0xxlcv000308jn6l70fcc7
slug: another-functional-options-pattern-in-golang-article
tags: golang, backend

---

In the world of Go (Golang), developers often face the challenge of creating flexible and expressive APIs while maintaining simplicity and readability. The Functional Options Pattern is a powerful technique that allows developers to design APIs that are both intuitive and extensible. In this article, we'll explore the advanced aspects of the Functional Options Pattern in Golang, accompanied by real-world examples to demonstrate its effectiveness.

## Basics of Functional Options Pattern:

The Functional Options Pattern involves using functional arguments to configure a type during its instantiation. It provides a clean and idiomatic way to customize the behavior of functions or constructors without relying on a multitude of parameters. Let's delve into the basics before exploring advanced usage.

```go
type Config struct {
    Option1 int
    Option2 string
}

type MyService struct {
    config Config
}

type Option func(*Config)

func WithOption1(value int) Option {
    return func(c *Config) {
        c.Option1 = value
    }
}

func WithOption2(value string) Option {
    return func(c *Config) {
        c.Option2 = value
    }
}

func NewMyService(options ...Option) *MyService {
    config := Config{
        Option1: 42,
        Option2: "default",
    }

    for _, option := range options {
        option(&config)
    }

    return &MyService{config: config}
}
```
This example demonstrates a simple configuration struct (Config), a service struct (MyService), and functional options (WithOption1 and WithOption2) to customize the configuration during instantiation.
## Examples:

### 1. Configuring Timeouts:
```go
type HttpClient struct {
    timeout time.Duration
}

type HttpClientOption func(*HttpClient)

func WithTimeout(timeout time.Duration) HttpClientOption {
    return func(c *HttpClient) {
        c.timeout = timeout
    }
}

func NewHttpClient(options ...HttpClientOption) *HttpClient {
    client := &HttpClient{
        timeout: 10 * time.Second, // default timeout
    }

    for _, option := range options {
        option(client)
    }

    return client
}
```
In this example, the HttpClient type is configured with a timeout. The WithTimeout option allows users to set a custom timeout when creating an instance of HttpClient.

### 2. Database Connection Pooling:
```go
type DatabaseConfig struct {
    maxConnections int
}

type DatabaseOption func(*DatabaseConfig)

func WithMaxConnections(max int) DatabaseOption {
    return func(c *DatabaseConfig) {
        c.maxConnections = max
    }
}

func NewDatabase(config DatabaseConfig, options ...DatabaseOption) *Database {
    // Initialize database with the provided configuration and options
}
```
Here, the Database type is configured with a maximum number of connections. Users can customize this value using the WithMaxConnections option when creating a new database instance.

### 3. Feature Flags:
```go
type FeatureService struct {
    enableFeatureX bool
}

type FeatureOption func(*FeatureService)

func EnableFeatureX() FeatureOption {
    return func(f *FeatureService) {
        f.enableFeatureX = true
    }
}

func NewFeatureService(options ...FeatureOption) *FeatureService {
    service := &FeatureService{
        enableFeatureX: false, // default is disabled
    }

    for _, option := range options {
        option(service)
    }

    return service
}
```
This example showcases how to use functional options for feature toggling. The EnableFeatureX option enables a specific feature when creating an instance of FeatureService.

## Conclusion:

The Functional Options Pattern in Golang is a powerful and expressive tool for designing flexible APIs. By providing a clean and concise way to configure types, developers can create APIs that are both easy to use and extend. The real-world examples presented here illustrate the versatility of this pattern, demonstrating its application in scenarios such as configuring timeouts, database connection pooling, and feature flags. As you embrace the Functional Options Pattern, remember that its elegance lies in its ability to balance simplicity and extensibility in your Go code.