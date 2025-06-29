# Fastify Architecture Diagram

## High-Level Architecture Overview

```mermaid
graph TB
    subgraph "Client Layer"
        Client[HTTP Client]
    end
    
    subgraph "Server Layer"
        HTTPServer[HTTP/HTTPS/HTTP2 Server]
    end
    
    subgraph "Core Fastify"
        Router[find-my-way Router]
        RequestLifecycle[Request Lifecycle]
        PluginSystem[Plugin System - Avvio]
        HookSystem[Hook System]
        Decorators[Decorators]
        SchemaController[Schema Controller]
        ContentParser[Content Type Parser]
        ErrorHandler[Error Handler]
        Logger[Logger - Pino]
    end
    
    subgraph "Request Processing"
        Request[Request Object]
        Reply[Reply Object]
        Context[Route Context]
        Validation[Validation]
        Serialization[Serialization]
    end
    
    Client -->|HTTP Request| HTTPServer
    HTTPServer -->|Route| Router
    Router -->|Create| Context
    Context -->|Initialize| Request
    Context -->|Initialize| Reply
    RequestLifecycle -->|Process| Request
    RequestLifecycle -->|Process| Reply
    
    PluginSystem -.->|Extends| Router
    PluginSystem -.->|Extends| HookSystem
    PluginSystem -.->|Extends| Decorators
```

## Detailed Component Architecture

```mermaid
graph LR
    subgraph "Main Entry (fastify.js)"
        FastifyFactory[Fastify Factory<br/>- Creates server instance<br/>- Configures options<br/>- Sets up plugins]
    end
    
    subgraph "Core Components"
        direction TB
        
        Server[Server Module<br/>lib/server.js<br/>- HTTP/HTTPS/HTTP2<br/>- Connection handling<br/>- Multiple bindings]
        
        Route[Route Module<br/>lib/route.js<br/>- Route registration<br/>- Method validation<br/>- Route configuration]
        
        Hooks[Hooks Module<br/>lib/hooks.js<br/>- Lifecycle hooks<br/>- Application hooks<br/>- Hook runners]
        
        Context[Context Module<br/>lib/context.js<br/>- Route context<br/>- Schema storage<br/>- Handler config]
        
        Request[Request Module<br/>lib/request.js<br/>- Request wrapper<br/>- Parameter parsing<br/>- Header handling]
        
        Reply[Reply Module<br/>lib/reply.js<br/>- Response wrapper<br/>- Serialization<br/>- Status/headers]
    end
    
    subgraph "Plugin & Extension System"
        Avvio[Avvio<br/>- Plugin loader<br/>- Dependency mgmt<br/>- Boot sequence]
        
        PluginUtils[Plugin Utils<br/>lib/pluginUtils.js<br/>- Plugin registration<br/>- Encapsulation]
        
        Decorate[Decorators<br/>lib/decorate.js<br/>- Add properties<br/>- Extend objects]
    end
    
    subgraph "Validation & Serialization"
        SchemaCtrl[Schema Controller<br/>lib/schema-controller.js<br/>- Schema storage<br/>- Compiler setup]
        
        Validation[Validation<br/>lib/validation.js<br/>- Request validation<br/>- Schema compilation]
        
        Schemas[Schemas<br/>lib/schemas.js<br/>- JSON Schema mgmt<br/>- Response schemas]
    end
    
    FastifyFactory --> Server
    FastifyFactory --> Route
    FastifyFactory --> Hooks
    FastifyFactory --> Avvio
```

## Request Lifecycle Flow

```mermaid
sequenceDiagram
    participant Client
    participant Server as HTTP Server
    participant Router as Router<br/>(find-my-way)
    participant Context as Route Context
    participant Hooks as Hook System
    participant Handler as Route Handler
    participant Reply as Reply Object
    
    Client->>Server: HTTP Request
    Server->>Router: Route Lookup
    Router->>Context: Get Route Context
    Context->>Hooks: Create Request/Reply
    
    Note over Hooks: Lifecycle Hooks
    
    Hooks->>Hooks: onRequest Hook
    Hooks->>Hooks: preParsing Hook
    Hooks->>Hooks: Parsing
    Hooks->>Hooks: preValidation Hook
    Hooks->>Hooks: Validation
    Hooks->>Hooks: preHandler Hook
    
    Hooks->>Handler: Execute Handler
    Handler->>Reply: Set Response
    
    Reply->>Hooks: preSerialization Hook
    Hooks->>Hooks: Serialization
    Hooks->>Hooks: onSend Hook
    
    Reply->>Client: HTTP Response
    Hooks->>Hooks: onResponse Hook
```

## Plugin System Architecture

```mermaid
graph TB
    subgraph "Plugin Encapsulation"
        RootContext[Root Context<br/>- Global decorations<br/>- Global hooks<br/>- Base configuration]
        
        Plugin1[Plugin Context 1<br/>- Isolated scope<br/>- Own decorations<br/>- Own hooks]
        
        Plugin2[Plugin Context 2<br/>- Isolated scope<br/>- Inherits from root<br/>- Cannot see Plugin 1]
        
        NestedPlugin[Nested Plugin<br/>- Inherits from Plugin 1<br/>- Extended scope]
    end
    
    RootContext --> Plugin1
    RootContext --> Plugin2
    Plugin1 --> NestedPlugin
    
    Note1[Encapsulation provides isolation<br/>between plugin contexts]
```

## Internal Module Dependencies

```mermaid
graph TD
    subgraph "Entry Points"
        Fastify[fastify.js<br/>Main Factory]
        Types[fastify.d.ts<br/>TypeScript Definitions]
    end
    
    subgraph "Core Request/Response"
        HandleRequest[handleRequest.js]
        Request[request.js]
        Reply[reply.js]
        Context[context.js]
    end
    
    subgraph "Routing & Validation"
        Route[route.js]
        FindMyWay[find-my-way<br/>External]
        Validation[validation.js]
        SchemaController[schema-controller.js]
    end
    
    subgraph "Plugin & Hook System"
        Hooks[hooks.js]
        Avvio[avvio<br/>External]
        PluginUtils[pluginUtils.js]
        PluginOverride[pluginOverride.js]
    end
    
    subgraph "Parsing & Serialization"
        ContentTypeParser[contentTypeParser.js]
        Schemas[schemas.js]
        AJV[ajv-compiler<br/>External]
        FastJSON[fast-json-stringify<br/>External]
    end
    
    subgraph "Server & Logging"
        Server[server.js]
        LoggerFactory[logger-factory.js]
        Pino[pino<br/>External]
    end
    
    subgraph "Error Handling"
        ErrorHandler[error-handler.js]
        Errors[errors.js]
        ErrorSerializer[error-serializer.js]
    end
    
    subgraph "Utilities"
        Symbols[symbols.js]
        Decorators[decorate.js]
        Warnings[warnings.js]
    end
    
    Fastify --> Server
    Fastify --> Route
    Fastify --> Hooks
    Fastify --> ContentTypeParser
    Fastify --> SchemaController
    Fastify --> Avvio
    
    Route --> FindMyWay
    Route --> Context
    Route --> HandleRequest
    Route --> Validation
    
    HandleRequest --> Request
    HandleRequest --> Reply
    HandleRequest --> Hooks
    
    Context --> Request
    Context --> Reply
    
    Validation --> AJV
    Validation --> FastJSON
    
    SchemaController --> Schemas
    
    Server --> LoggerFactory
    LoggerFactory --> Pino
    
    ErrorHandler --> Errors
    ErrorHandler --> ErrorSerializer
```

## Key Architectural Components

### 1. **Core Server (lib/server.js)**
- Handles HTTP/HTTPS/HTTP2 server creation
- Manages multiple network bindings
- Handles server lifecycle (listen, close)
- Connection management

### 2. **Routing System (lib/route.js + find-my-way)**
- High-performance router using find-my-way
- Supports parametric routes
- Constraint-based routing
- Method validation

### 3. **Hook System (lib/hooks.js)**
- **Lifecycle Hooks**: onRequest, preParsing, preValidation, preHandler, preSerialization, onSend, onResponse, onError
- **Application Hooks**: onRoute, onRegister, onReady, onListen, preClose, onClose
- Async/sync hook support
- Hook inheritance through encapsulation

### 4. **Plugin System (Avvio)**
- Encapsulated plugin contexts
- Dependency management
- Asynchronous boot sequence
- Plugin isolation

### 5. **Content Type Parser (lib/contentTypeParser.js)**
- Pluggable parsers for different content types
- Built-in parsers for JSON, text
- Custom parser support
- Body size limiting

### 6. **Schema Validation System**
- JSON Schema support
- Request/response validation
- Compiler plugins (AJV default)
- Serialization optimization

### 7. **Decorator System (lib/decorate.js)**
- Extend Fastify instance
- Extend Request/Reply objects
- Encapsulation-aware

### 8. **Error Handling (lib/error-handler.js)**
- Centralized error handling
- Custom error handlers per route
- Error serialization
- Schema validation errors

### 9. **Logging (lib/logger.js + Pino)**
- High-performance structured logging
- Request/response logging
- Child loggers per request
- Custom serializers

## Data Flow Patterns

```mermaid
graph LR
    subgraph "Request Flow"
        A[Incoming Request] --> B[Route Matching]
        B --> C[Context Creation]
        C --> D[Hook Pipeline]
        D --> E[Handler Execution]
        E --> F[Response Generation]
    end
    
    subgraph "Plugin Registration"
        G[Register Plugin] --> H[Create Context]
        H --> I[Run Plugin Code]
        I --> J[Apply Decorations]
        J --> K[Register Routes]
    end
    
    subgraph "Schema Compilation"
        L[Define Schema] --> M[Compile Validator]
        M --> N[Cache Compiled]
        N --> O[Runtime Validation]
    end
```

## Key Design Principles

1. **High Performance**: Optimized for speed with careful hot-path optimization
2. **Encapsulation**: Plugin contexts provide isolation and prevent conflicts
3. **Extensibility**: Decorators and hooks allow extending functionality
4. **Schema-First**: JSON Schema validation built into the core
5. **Asynchronous**: Full async/await and promise support
6. **Low Overhead**: Minimal abstractions in the request path

## File Structure Mapping

- **Entry Point**: `fastify.js`
- **Core Modules**: `lib/`
  - Request/Reply handling
  - Routing and hooks
  - Plugin system integration
- **Type Definitions**: `types/` (TypeScript support)
- **Tests**: `test/` (comprehensive test suite)
- **Documentation**: `docs/` (guides and references)

This architecture enables Fastify to be both highly performant and developer-friendly, with a plugin system that promotes code reusability and maintainability.