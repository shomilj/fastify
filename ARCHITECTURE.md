# Fastify Architecture

## Overview

Fastify is a high-performance web framework for Node.js built on a modular architecture with a focus on developer experience and low overhead. This document provides architectural diagrams and explanations of Fastify's core components and flows.

## Table of Contents

1. [High-Level Architecture](#high-level-architecture)
2. [Initialization Flow](#initialization-flow)
3. [Request Lifecycle](#request-lifecycle)
4. [Component Architecture](#component-architecture)
5. [Plugin System](#plugin-system)
6. [Hooks System](#hooks-system)

## High-Level Architecture

```mermaid
graph TB
    subgraph "Fastify Core"
        A[fastify.js<br/>Main Entry Point] --> B[Server Creation]
        B --> C[HTTP/HTTPS/HTTP2 Server]
        
        A --> D[Plugin System<br/>Avvio]
        A --> E[Router<br/>find-my-way]
        A --> F[Schema Controller]
        A --> G[Hooks System]
        A --> H[Content Type Parser]
        
        subgraph "Core Libraries"
            I[lib/server.js<br/>Server Factory]
            J[lib/route.js<br/>Route Management]
            K[lib/context.js<br/>Encapsulation Context]
            L[lib/request.js<br/>Request Object]
            M[lib/reply.js<br/>Reply Object]
            N[lib/hooks.js<br/>Hook Management]
            O[lib/contentTypeParser.js<br/>Body Parsing]
            P[lib/validation.js<br/>Schema Validation]
            Q[lib/errors.js<br/>Error Handling]
        end
        
        D --> K
        E --> J
        F --> P
        G --> N
        H --> O
    end
    
    subgraph "External Dependencies"
        R[pino<br/>Logger]
        S[ajv<br/>JSON Schema Validator]
        T[fast-json-stringify<br/>Serializer]
        U[find-my-way<br/>Router]
        V[avvio<br/>Plugin Boot]
    end
    
    A --> R
    F --> S
    F --> T
    E --> U
    D --> V
```

## Initialization Flow

```mermaid
sequenceDiagram
    participant User
    participant Fastify
    participant Avvio
    participant Router
    participant Server
    participant Hooks
    
    User->>Fastify: fastify(options)
    activate Fastify
    
    Fastify->>Fastify: Validate options
    Fastify->>Fastify: Create logger
    Fastify->>Router: Build routing (find-my-way)
    Fastify->>Server: Create HTTP/HTTPS/HTTP2 server
    Fastify->>Hooks: Initialize hooks
    Fastify->>Avvio: Setup plugin system
    
    Note over Fastify: Register core decorators
    Fastify->>Fastify: decorate()
    Fastify->>Fastify: decorateRequest()
    Fastify->>Fastify: decorateReply()
    
    Fastify-->>User: Return fastify instance
    deactivate Fastify
    
    User->>Fastify: register(plugin)
    activate Fastify
    Fastify->>Avvio: Load plugin
    Avvio->>Avvio: Create new context
    Avvio->>Plugin: Execute plugin
    Plugin->>Fastify: Add routes/hooks/decorators
    Avvio-->>Fastify: Plugin loaded
    deactivate Fastify
    
    User->>Fastify: listen(port)
    activate Fastify
    Fastify->>Avvio: Boot application
    Avvio->>Hooks: Run onReady hooks
    Avvio->>Server: Start listening
    Server->>Hooks: Run onListen hooks
    Fastify-->>User: Server started
    deactivate Fastify
```

## Request Lifecycle

```mermaid
flowchart TB
    Start([HTTP Request]) --> Router{Route<br/>Matching}
    
    Router -->|Found| Context[Load Route Context]
    Router -->|Not Found| FourOhFour[404 Handler]
    
    Context --> CreateReq[Create Request Object]
    CreateReq --> CreateReply[Create Reply Object]
    
    CreateReply --> OnRequest{onRequest<br/>Hook}
    OnRequest -->|Error| ErrorHandler
    OnRequest -->|Success| PreParsing{preParsing<br/>Hook}
    
    PreParsing -->|Error| ErrorHandler
    PreParsing -->|Success| ParseBody[Content Type Parser]
    
    ParseBody -->|Error| ErrorHandler
    ParseBody -->|Success| PreValidation{preValidation<br/>Hook}
    
    PreValidation -->|Error| ErrorHandler
    PreValidation -->|Success| Validation[Schema Validation]
    
    Validation -->|Error| ValidationError{attachValidation?}
    ValidationError -->|false| ErrorHandler
    ValidationError -->|true| AttachError[Attach Error to Request]
    
    Validation -->|Success| PreHandler{preHandler<br/>Hook}
    AttachError --> PreHandler
    
    PreHandler -->|Error| ErrorHandler
    PreHandler -->|Success| Handler[Route Handler]
    
    Handler -->|Error| ErrorHandler
    Handler -->|Success| PreSerialization{preSerialization<br/>Hook}
    
    PreSerialization -->|Error| ErrorHandler
    PreSerialization -->|Success| Serialize[Response Serialization]
    
    Serialize --> OnSend{onSend<br/>Hook}
    
    OnSend -->|Error| ErrorHandler
    OnSend -->|Success| Send[Send Response]
    
    Send --> OnResponse{onResponse<br/>Hook}
    
    ErrorHandler[Error Handler] --> OnError{onError<br/>Hook}
    OnError --> Send
    
    FourOhFour --> Send
    
    OnResponse --> End([Response Sent])
    
    style Start fill:#e1f5e1
    style End fill:#ffe1e1
    style ErrorHandler fill:#ffcccc
```

## Component Architecture

```mermaid
graph TB
    subgraph "Request/Reply Objects"
        Request[Request<br/>- id<br/>- params<br/>- query<br/>- headers<br/>- body<br/>- raw]
        Reply[Reply<br/>- statusCode<br/>- headers<br/>- sent<br/>- raw]
    end
    
    subgraph "Context System"
        Context[Context<br/>- schema<br/>- handler<br/>- errorHandler<br/>- hooks<br/>- config]
        Encapsulation[Encapsulation<br/>- Isolated plugin contexts<br/>- Inheritance chain]
    end
    
    subgraph "Schema Management"
        SchemaController[Schema Controller<br/>- Validator Compiler<br/>- Serializer Compiler<br/>- Schema Storage]
        Validation[Validation<br/>- Body<br/>- Headers<br/>- Query<br/>- Params]
        Serialization[Serialization<br/>- Response schemas<br/>- Fast JSON stringify]
    end
    
    subgraph "Content Handling"
        ContentTypeParser[Content Type Parser<br/>- JSON<br/>- Text<br/>- Custom parsers]
        BodyLimit[Body Limit<br/>- Global limit<br/>- Route-specific limit]
    end
    
    subgraph "Error Management"
        ErrorHandler[Error Handler<br/>- Default handler<br/>- Custom handlers<br/>- Error serialization]
        Errors[Error Codes<br/>- FST_ERR_*<br/>- Typed errors]
    end
    
    subgraph "Logging"
        Logger[Logger<br/>- Pino integration<br/>- Child loggers<br/>- Serializers]
    end
    
    Context --> Request
    Context --> Reply
    Context --> SchemaController
    Context --> ErrorHandler
    
    Request --> ContentTypeParser
    ContentTypeParser --> BodyLimit
    
    SchemaController --> Validation
    SchemaController --> Serialization
    
    ErrorHandler --> Errors
    Request --> Logger
    Reply --> Logger