# Fastify Architecture Diagram

## Overview

Fastify is a high-performance web framework for Node.js, designed with a plugin-based architecture and an emphasis on developer experience. This document provides a comprehensive architectural overview of the Fastify framework.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                            Fastify Application                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │   Server    │  │   Router    │  │   Plugins   │  │  Decorators  │  │
│  │  (HTTP/S)   │  │(find-my-way)│  │   (Avvio)   │  │              │  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘  │
│         │                 │                 │                │          │
│  ┌──────┴─────────────────┴─────────────────┴────────────────┴──────┐  │
│  │                      Core Engine (fastify.js)                    │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Entry Point & Initialization

```
fastify.js
    │
    ├─> Creates Fastify Instance
    │   ├─> HTTP/HTTPS/HTTP2 Server (lib/server.js)
    │   ├─> Router Instance (find-my-way)
    │   ├─> Plugin System (Avvio)
    │   ├─> Hooks System (lib/hooks.js)
    │   ├─> Schema Controller (lib/schema-controller.js)
    │   └─> Content Type Parser (lib/contentTypeParser.js)
    │
    └─> Returns Fastify API
```

### 2. Request Lifecycle

```
┌─────────────────┐
│ Incoming Request│
└────────┬────────┘
         │
         v
┌─────────────────┐     ┌──────────────┐
│  HTTP Handler   │────>│ Route Lookup │
│(lib/server.js)  │     │(find-my-way) │
└────────┬────────┘     └──────┬───────┘
         │                     │
         v                     v
┌─────────────────┐     ┌──────────────┐
│ Route Handler   │<────│Route Context │
│(lib/route.js)   │     │(lib/context) │
└────────┬────────┘     └──────────────┘
         │
         v
┌─────────────────────────────────────┐
│         Hooks Pipeline              │
│  ┌────────────────────────────┐    │
│  │ 1. onRequest               │    │
│  │ 2. preParsing              │    │
│  │ 3. preValidation           │    │
│  │ 4. preHandler              │    │
│  │ 5. Route Handler           │    │
│  │ 6. preSerialization        │    │
│  │ 7. onSend                  │    │
│  │ 8. onResponse              │    │
│  └────────────────────────────┘    │
└────────┬────────────────────────────┘
         │
         v
┌─────────────────┐
│ Response Sent   │
└─────────────────┘
```

### 3. Plugin Architecture (Encapsulation)

```
┌─────────────────────────────────────────────────┐
│                Root Context                     │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │ Plugin A  │  │ Plugin B  │  │ Plugin C  │  │
│  │           │  │           │  │           │  │
│  │ ┌───────┐ │  │ ┌───────┐ │  │ ┌───────┐ │  │
│  │ │Child 1│ │  │ │Child 1│ │  │ │Child 1│ │  │
│  │ └───────┘ │  │ └───────┘ │  │ └───────┘ │  │
│  │ ┌───────┐ │  │           │  │ ┌───────┐ │  │
│  │ │Child 2│ │  │           │  │ │Child 2│ │  │
│  │ └───────┘ │  │           │  │ └───────┘ │  │
│  └───────────┘  └───────────┘  └───────────┘  │
└─────────────────────────────────────────────────┘

Each plugin creates an encapsulated context with:
- Own decorators
- Own hooks
- Own routes
- Own error handlers
```

### 4. Core Libraries Structure

```
lib/
├── server.js           # HTTP/HTTPS/HTTP2 server creation
├── route.js            # Route registration and handling
├── hooks.js            # Lifecycle hooks implementation
├── context.js          # Request context management
├── request.js          # Request object factory
├── reply.js            # Reply object factory
├── contentTypeParser.js # Body parsing logic
├── schema-controller.js # JSON Schema validation/serialization
├── pluginUtils.js      # Plugin management utilities
├── pluginOverride.js   # Encapsulation logic
├── decorate.js         # Decorator pattern implementation
├── errors.js           # Error definitions
├── error-handler.js    # Error handling logic
├── fourOhFour.js       # 404 handling
├── handleRequest.js    # Main request handler
├── validation.js       # Schema validation logic
├── symbols.js          # Internal symbols
└── logger-factory.js   # Logger creation
```

## Key Architectural Patterns

### 1. Encapsulation & Inheritance

```
┌─────────────────┐
│  Parent Context │
│  - decorators   │
│  - hooks        │
│  - schemas      │
└────────┬────────┘
         │ inherits
         v
┌─────────────────┐
│  Child Context  │
│  - decorators + │
│  - hooks +      │
│  - schemas +    │
└─────────────────┘
```

### 2. Hook System

```
Application Hooks:
- onRoute       # When route is registered
- onRegister    # When plugin is registered  
- onReady       # When server is ready
- onListen      # When server starts listening
- preClose      # Before server closes
- onClose       # When server closes

Lifecycle Hooks (per request):
- onTimeout     # Request timeout
- onRequest     # Start of request
- preParsing    # Before parsing body
- preValidation # Before validation
- preHandler    # Before route handler
- preSerialization # Before serialization
- onSend        # Before sending response
- onResponse    # After response sent
- onError       # On error
- onRequestAbort # Request aborted
```

### 3. Validation & Serialization Flow

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│   Request   │────>│ Schema Validator │────>│   Handler   │
│    Body     │     │  (AJV Compiler)  │     │             │
└─────────────┘     └──────────────────┘     └──────┬───────┘
                                                     │
                                                     v
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│  Response   │<────│Schema Serializer │<────│   Result    │
│             │     │ (fast-json-      │     │             │
│             │     │  stringify)      │     │             │
└─────────────┘     └──────────────────┘     └─────────────┘
```

### 4. Content Type Parsing

```
┌─────────────────────┐
│  Content-Type Header│
└──────────┬──────────┘
           │
           v
┌─────────────────────┐     ┌──────────────┐
│ ContentTypeParser   │────>│ Parser Func  │
│  Registry           │     │ (registered) │
└─────────────────────┘     └──────┬───────┘
                                   │
                                   v
                            ┌──────────────┐
                            │ Parsed Body  │
                            └──────────────┘
```

## Type System Integration

```
types/
├── fastify.d.ts        # Main type definitions
├── instance.d.ts       # Fastify instance types
├── request.d.ts        # Request types
├── reply.d.ts          # Reply types
├── route.d.ts          # Route types
├── hooks.d.ts          # Hook types
├── schema.d.ts         # Schema types
├── plugin.d.ts         # Plugin types
├── type-provider.d.ts  # Type provider system
└── ...
```

## Performance Optimizations

1. **Route Lookup**: Uses radix tree (find-my-way) for O(k) lookups
2. **Schema Compilation**: Pre-compiles validators and serializers
3. **Lazy Loading**: Components loaded only when needed
4. **Object Pooling**: Reuses request/reply objects
5. **Async/Await Support**: Native promise handling
6. **Stream Support**: Efficient handling of streams

## External Dependencies

```
Core Dependencies:
- avvio                 # Plugin system
- find-my-way          # Router
- pino                 # Logger
- @fastify/ajv-compiler # Schema validation
- @fastify/fast-json-stringify-compiler # Serialization
- light-my-request      # Request injection (testing)
```

## Extension Points

1. **Decorators**: Extend request/reply/instance objects
2. **Hooks**: Intercept request lifecycle
3. **Plugins**: Encapsulated functionality
4. **Content Type Parsers**: Custom body parsing
5. **Schema Compilers**: Custom validation/serialization
6. **Error Handlers**: Custom error handling
7. **Constraints**: Custom routing constraints

## Summary

Fastify's architecture is built around:
- **Performance**: Optimized for high throughput
- **Extensibility**: Plugin-based architecture with encapsulation
- **Developer Experience**: Comprehensive validation, serialization, and error handling
- **Type Safety**: Full TypeScript support with type providers
- **Standards**: Built on web standards with HTTP/2 support

The framework achieves its high performance through careful optimization of the request lifecycle, efficient routing, and pre-compilation of schemas while maintaining a clean, extensible architecture.