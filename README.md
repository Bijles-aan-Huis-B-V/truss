# Truss (Deprecated Fork)

> **This is a deprecated fork.** The original repository is no longer maintained. We plan to archive this once booking-service, user-service, and translation-service are migrated to Echo.

## Overview

Go CLI tool that generates gRPC microservices with go-kit from Protocol Buffer definitions. Users define services in `.proto` files and Truss generates a complete service scaffold including gRPC/HTTP transports, endpoints, clients, and handler stubs. Key feature: smart handler preservation — user-written business logic survives regeneration.

## Why We're Moving Away

While the concept of generating both proto and HTTP services from a single proto file is elegant in theory, in practice it has significant drawbacks:
- Reading from HTTP headers or cookies is extremely painful
- Maintenance burden with no upstream support
- Single handler function per endpoint limits flexibility
- Generated code is hard to debug

## Services Still Using Truss

- booking-service
- user-service
- translation-service

## Build

```bash
make                    # Build truss
make test               # Run all tests
```
