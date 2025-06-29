# Fastify Architecture Diagram

## Overview
Fastify is a high-performance web framework for Node.js with a plugin-based architecture, comprehensive lifecycle hooks, and built-in schema validation.

## Core Architecture

```mermaid
graph TB
    subgraph "Entry Point"
        MAIN[fastify.js<br/>Main Entry Point]
    end

    subgraph "Core Components"
        AVVIO[Avvio<br/>Plugin System]
        ROUTER[find-my-way<br/>Router]
        SERVER[HTTP/HTTPS/HTTP2<br/>Server]
    end

    subgraph "Request Pipeline"
        REQ[Request Object]
        REPLY[Reply Object]
        HANDLER[Route Handler]
        CONTEXT[Route Context]
    end

    subgraph "Lifecycle Hooks"
        HOOKS[Hooks System]
        APPHOOKS[Application Hooks<br/>- onRoute<br/>- onRegister<br/>- onReady<br/>- onListen<br/>- preClose<br/>- onClose]
        REQHOOKS[Request Hooks<br/>- onRequest<br/>- preParsing<br/>- preValidation<br/>- preHandler<br/>- preSerialization<br/>- onSend<br/>- onResponse<br/>- onError<br/>- onTimeout<br/>- onRequestAbort]
    end

    subgraph "Validation & Serialization"
        SCHEMA[Schema Controller]
        AJV[AJV Validator]
        FJS[fast-json-stringify]
    end

    subgraph "Content Handling"
        CTP[Content Type Parser]
        PARSERS[Built-in Parsers<br/>- JSON<br/>- Text<br/>- Buffer<br/>- Stream]
    end

    subgraph "Error Management"
        ERROR[Error Handler]
        FOUROFOUR[404 Handler]
        ERRTYPES[Error Types<br/>- Validation Errors<br/>- Runtime Errors<br/>- Plugin Errors]
    end

    subgraph "Logging"
        LOGGER[Pino Logger]
        CHILDLOG[Child Loggers<br/>per Request]
    end

    subgraph "Decorators & Encapsulation"
        DECORATOR[Decorator System]
        ENCAP[Context Encapsulation]
    end

    MAIN --> AVVIO
    MAIN --> ROUTER
    MAIN --> SERVER
    
    SERVER --> REQ
    REQ --> CONTEXT
    CONTEXT --> HOOKS
    
    HOOKS --> APPHOOKS
    HOOKS --> REQHOOKS
    
    REQHOOKS --> CTP
    CTP --> PARSERS
    
    REQHOOKS --> SCHEMA
    SCHEMA --> AJV
    SCHEMA --> FJS
    
    CONTEXT --> HANDLER
    HANDLER --> REPLY
    
    REPLY --> REQHOOKS
    
    ERROR --> FOUROFOUR
    ERROR --> ERRTYPES
    
    REQ --> CHILDLOG
    CHILDLOG --> LOGGER
    
    AVVIO --> DECORATOR
    DECORATOR --> ENCAP
    ENCAP --> CONTEXT

    style MAIN fill:#f9f,stroke:#333,stroke-width:4px
    style SERVER fill:#bbf,stroke:#333,stroke-width:2px
    style HOOKS fill:#bfb,stroke:#333,stroke-width:2px
    style SCHEMA fill:#fbf,stroke:#333,stroke-width:2px
```

## Component Details

### 1. **Entry Point (`fastify.js`)**
- Creates the Fastify instance
- Initializes all core components
- Exposes public API methods
- Sets up default configurations

### 2. **Plugin System (Avvio)**
- Manages asynchronous plugin registration
- Provides encapsulation between plugins
- Handles plugin dependencies
- Supports plugin metadata and versioning

### 3. **Routing (find-my-way)**
- High-performance HTTP router
- Supports parametric and wildcard routes
- Constraint-based routing (versioning, host, etc.)
- Method-based routing

### 4. **Server Layer**
- Supports HTTP/1.1, HTTP/2, and HTTPS
- Configurable timeouts and limits
- Connection management
- Request/Response handling

### 5. **Request Lifecycle**

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Router
    participant Hooks
    participant ContentParser
    participant Validator
    participant Handler
    participant Serializer
    participant Reply

    Client->>Server: HTTP Request
    Server->>Router: Route Lookup
    Router->>Hooks: onRequest Hook
    Hooks->>ContentParser: preParsing Hook
    ContentParser->>ContentParser: Parse Body
    ContentParser->>Validator: preValidation Hook
    Validator->>Validator: Validate Schema
    Validator->>Hooks: preHandler Hook
    Hooks->>Handler: Execute Handler
    Handler->>Reply: Generate Response
    Reply->>Hooks: preSerialization Hook
    Hooks->>Serializer: Serialize Response
    Serializer->>Hooks: onSend Hook
    Hooks->>Client: Send Response
    Hooks->>Hooks: onResponse Hook
```

### 6. **Hook System**
- **Application Hooks**: Control server lifecycle
- **Request Hooks**: Intercept request/response flow
- Supports both sync and async hooks
- Error propagation through hook chain

### 7. **Validation & Serialization**
- JSON Schema validation via AJV
- Fast JSON serialization via fast-json-stringify
- Compiled schemas for performance
- Custom error formatting

### 8. **Content Type Parsing**
- Pluggable parser system
- Built-in parsers for common types
- Custom parser registration
- Body size limits

### 9. **Context & Encapsulation**
- Each route has its own context
- Encapsulated decorators per plugin
- Inherited from parent contexts
- Isolated configuration

### 10. **Error Handling**
- Centralized error handler
- Custom 404 handling per context
- Error serialization
- Hook-based error interception

### 11. **Logging**
- Based on Pino (high-performance logger)
- Request-scoped child loggers
- Configurable log levels
- Custom serializers

## Key Features

### Performance Optimizations
- Route compilation at startup
- Schema compilation for validation/serialization
- Minimal overhead in request handling
- Efficient plugin loading

### Developer Experience
- Chainable API
- TypeScript support
- Comprehensive error messages
- Extensive plugin ecosystem

### Security Features
- Prototype poisoning protection
- Request timeout handling
- Body size limits
- Schema validation

## File Structure Mapping

```
fastify/
├── fastify.js              # Main entry point
├── lib/
│   ├── server.js           # HTTP server creation
│   ├── route.js            # Route registration
│   ├── context.js          # Route context
│   ├── hooks.js            # Hook system
│   ├── request.js          # Request object
│   ├── reply.js            # Reply object
│   ├── contentTypeParser.js # Content parsing
│   ├── schema-controller.js # Schema management
│   ├── validation.js       # Schema validation
│   ├── handleRequest.js    # Request handler
│   ├── pluginUtils.js      # Plugin utilities
│   ├── pluginOverride.js   # Plugin encapsulation
│   ├── decorate.js         # Decorator system
│   ├── errors.js           # Error definitions
│   ├── error-handler.js    # Error handling
│   ├── fourOhFour.js       # 404 handling
│   ├── logger-factory.js   # Logger creation
│   └── symbols.js          # Internal symbols
├── types/                  # TypeScript definitions
└── test/                   # Test suite
```

## Plugin Architecture

```mermaid
graph LR
    subgraph "Plugin Registration"
        REGISTER[fastify.register]
        META[Plugin Metadata]
        DEPS[Dependencies Check]
        VER[Version Check]
    end

    subgraph "Plugin Context"
        PARENT[Parent Context]
        CHILD[Child Context]
        DEC[Decorators]
        HOOKS2[Hooks]
    end

    REGISTER --> META
    META --> DEPS
    DEPS --> VER
    VER --> CHILD
    PARENT -.inherit.-> CHILD
    CHILD --> DEC
    CHILD --> HOOKS2
```

## Request Flow Summary

1. **Server receives request** → HTTP handler
2. **Router finds route** → find-my-way lookup
3. **Create request context** → Request/Reply objects
4. **Run lifecycle hooks** → Sequential hook execution
5. **Parse body** → Content-Type based parsing
6. **Validate request** → JSON Schema validation
7. **Execute handler** → User route handler
8. **Serialize response** → fast-json-stringify
9. **Send response** → Reply.send()
10. **Log and cleanup** → onResponse hooks

This architecture enables Fastify to achieve high performance while maintaining a clean, extensible design with excellent developer experience.