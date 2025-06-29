# Fastify Architecture Diagrams

This document provides comprehensive architecture diagrams for the Fastify web framework, illustrating its core components, request lifecycle, plugin system, and overall structure.

## Table of Contents
- [High-Level Architecture Overview](#high-level-architecture-overview)
- [Core Components](#core-components)
- [Request Lifecycle](#request-lifecycle)
- [Plugin System & Encapsulation](#plugin-system--encapsulation)
- [Detailed Component Interactions](#detailed-component-interactions)

## High-Level Architecture Overview

```mermaid
graph TB
    subgraph "Fastify Application"
        A[HTTP Server<br/>node:http/https] --> B[Fastify Core]
        
        subgraph "Core Components"
            B --> C[Router<br/>find-my-way]
            B --> D[Avvio<br/>Plugin System]
            B --> E[Pino<br/>Logger]
            B --> F[Schema Controller<br/>Ajv]
            B --> G[Content Type Parser]
            B --> H[Hooks System]
        end
        
        subgraph "Request Processing"
            I[Request Handler] --> J[Request Object]
            I --> K[Reply Object]
            I --> L[Context]
        end
        
        subgraph "Plugin Architecture"
            M[Root Context] --> N[Child Context 1]
            M --> O[Child Context 2]
            N --> P[Grandchild Context]
        end
    end
    
    Client[HTTP Client] --> A
    C --> I
    D --> M
    H --> I
```

## Core Components

```mermaid
graph LR
    subgraph "Entry Point"
        FJ[fastify.js] --> |creates| S[Server]
    end
    
    subgraph "Core Libraries"
        FJ --> CTP[ContentTypeParser]
        FJ --> SC[SchemaController]
        FJ --> CTX[Context]
        FJ --> DEC[Decorator]
        FJ --> ERR[Error Handler]
        FJ --> HKS[Hooks]
        FJ --> RT[Route]
    end
    
    subgraph "Infrastructure"
        S --> |uses| SRV[lib/server.js]
        SRV --> |creates| HTTP[HTTP/HTTPS/HTTP2]
        FJ --> |logger| LOG[Logger Factory]
        LOG --> PINO[Pino Logger]
    end
    
    subgraph "Request/Reply"
        RT --> REQ[Request<br/>lib/request.js]
        RT --> REP[Reply<br/>lib/reply.js]
        REQ --> |validation| VAL[Validation<br/>lib/validation.js]
        REP --> |serialization| SER[Schemas<br/>lib/schemas.js]
    end
    
    subgraph "Plugin System"
        FJ --> |powered by| AVV[Avvio]
        AVV --> PLG[Plugin Utils]
        PLG --> OVR[Plugin Override]
    end
    
    subgraph "Routing"
        RT --> |404 handling| FOF[FourOhFour<br/>lib/fourOhFour.js]
        RT --> |request handling| HND[HandleRequest<br/>lib/handleRequest.js]
    end
```

## Request Lifecycle

```mermaid
flowchart TD
    START[Incoming Request] --> ROUTE[Routing]
    ROUTE --> LOGGER[Instance Logger]
    LOGGER --> ONREQ[onRequest Hook]
    
    ONREQ --> |success| PREPARSE[preParsing Hook]
    ONREQ --> |error| ERR1[4xx/5xx Response]
    
    PREPARSE --> |success| PARSE[Body Parsing]
    PREPARSE --> |error| ERR2[4xx/5xx Response]
    
    PARSE --> |success| PREVAL[preValidation Hook]
    PARSE --> |error| ERR3[4xx/5xx Response]
    
    PREVAL --> |success| VAL[Schema Validation]
    PREVAL --> |error| ERR4[4xx/5xx Response]
    
    VAL --> |success| PREHAND[preHandler Hook]
    VAL --> |error| ERR5[400 Bad Request]
    
    PREHAND --> |success| HANDLER[User Handler]
    PREHAND --> |error| ERR6[4xx/5xx Response]
    
    HANDLER --> |success| REPLY[Reply]
    HANDLER --> |error| ERR7[4xx/5xx Response]
    
    REPLY --> PRESER[preSerialization Hook]
    
    PRESER --> |success| ONSEND[onSend Hook]
    PRESER --> |error| ERR8[4xx/5xx Response]
    
    ONSEND --> |success| RESP[Outgoing Response]
    ONSEND --> |error| ERR9[4xx/5xx Response]
    
    RESP --> ONRESP[onResponse Hook]
    
    style ONREQ fill:#e1f5e1
    style PREPARSE fill:#e1f5e1
    style PREVAL fill:#e1f5e1
    style PREHAND fill:#e1f5e1
    style PRESER fill:#e1f5e1
    style ONSEND fill:#e1f5e1
    style ONRESP fill:#e1f5e1
    style HANDLER fill:#ffe1e1
    style PARSE fill:#e1e1ff
    style VAL fill:#e1e1ff
```

## Plugin System & Encapsulation

```mermaid
graph TD
    subgraph "Root Context"
        ROOT[Root Fastify Instance]
        ROOT_DEC[Root Decorators]
        ROOT_HOOKS[Root Hooks]
        ROOT_ROUTES[Root Routes]
    end
    
    subgraph "Plugin 1 Context"
        P1[Plugin 1]
        P1_DEC[Plugin 1 Decorators]
        P1_HOOKS[Plugin 1 Hooks]
        P1_ROUTES[Plugin 1 Routes]
    end
    
    subgraph "Plugin 2 Context"
        P2[Plugin 2]
        P2_DEC[Plugin 2 Decorators]
        P2_HOOKS[Plugin 2 Hooks]
        P2_ROUTES[Plugin 2 Routes]
    end
    
    subgraph "Child Plugin Context"
        P1_1[Child Plugin 1.1]
        P1_1_DEC[Plugin 1.1 Decorators]
        P1_1_HOOKS[Plugin 1.1 Hooks]
        P1_1_ROUTES[Plugin 1.1 Routes]
    end
    
    ROOT --> |register| P1
    ROOT --> |register| P2
    P1 --> |register| P1_1
    
    P1 -.->|inherits| ROOT_DEC
    P1 -.->|inherits| ROOT_HOOKS
    P2 -.->|inherits| ROOT_DEC
    P2 -.->|inherits| ROOT_HOOKS
    
    P1_1 -.->|inherits| P1_DEC
    P1_1 -.->|inherits| P1_HOOKS
    P1_1 -.->|inherits| ROOT_DEC
    P1_1 -.->|inherits| ROOT_HOOKS
    
    P1 -.-x|no access| P2_DEC
    P1 -.-x|no access| P1_1_DEC
    ROOT -.-x|no access| P1_DEC
    ROOT -.-x|no access| P2_DEC
    
    style ROOT fill:#ffd4d4
    style P1 fill:#d4ffd4
    style P2 fill:#d4d4ff
    style P1_1 fill:#ffffd4
```

## Detailed Component Interactions

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
    Server->>Router: Route lookup
    Router->>Hooks: Execute onRequest
    
    alt onRequest success
        Hooks->>Hooks: Execute preParsing
        Hooks->>ContentParser: Parse body
        ContentParser->>Hooks: Body parsed
        Hooks->>Hooks: Execute preValidation
        Hooks->>Validator: Validate request
        
        alt Validation success
            Validator->>Hooks: Valid
            Hooks->>Hooks: Execute preHandler
            Hooks->>Handler: Call user handler
            Handler->>Reply: Generate response
            Reply->>Hooks: Execute preSerialization
            Hooks->>Serializer: Serialize response
            Serializer->>Hooks: Serialized
            Hooks->>Hooks: Execute onSend
            Hooks->>Server: Send response
            Server->>Client: HTTP Response
            Server->>Hooks: Execute onResponse
        else Validation failure
            Validator->>Reply: 400 Bad Request
            Reply->>Server: Error response
            Server->>Client: HTTP Error
        end
    else Hook error
        Hooks->>Reply: Error response
        Reply->>Server: Send error
        Server->>Client: HTTP Error
    end
```

## Key Architecture Characteristics

### 1. **Plugin-Based Architecture**
- Everything is a plugin (routes, decorators, hooks)
- Encapsulated contexts prevent pollution
- Inheritance flows down, not up
- `fastify-plugin` can break encapsulation when needed

### 2. **High Performance Design**
- Efficient routing with find-my-way
- Schema-based validation and serialization
- Minimal overhead in hot paths
- Streaming support

### 3. **Lifecycle Hooks**
- Request/Reply hooks for fine-grained control
- Application hooks for lifecycle management
- Asynchronous hook support
- Error propagation through hook chain

### 4. **Schema-First Approach**
- JSON Schema validation with Ajv
- Automatic serialization optimization
- Type inference support
- Custom schema formats

### 5. **Extensibility**
- Decorators for extending core objects
- Custom content type parsers
- Plugin system via Avvio
- Custom error handlers

### 6. **Logging**
- Pino logger integration
- Request ID tracking
- Child logger support
- Serializer customization

## File Structure Mapping

```
fastify/
├── fastify.js              # Main entry point
├── lib/
│   ├── server.js           # HTTP server creation
│   ├── route.js            # Routing logic
│   ├── request.js          # Request object
│   ├── reply.js            # Reply object
│   ├── context.js          # Encapsulation context
│   ├── hooks.js            # Hooks system
│   ├── contentTypeParser.js # Body parsing
│   ├── schema-controller.js # Schema management
│   ├── validation.js       # Request validation
│   ├── decorate.js         # Decorator system
│   ├── pluginUtils.js      # Plugin utilities
│   ├── logger-factory.js   # Logger creation
│   └── errors.js           # Error definitions
└── types/                  # TypeScript definitions
```

This architecture enables Fastify to achieve high performance while maintaining developer-friendly APIs and strong extensibility through its plugin system.