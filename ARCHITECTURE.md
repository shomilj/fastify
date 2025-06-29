# Fastify Architecture Diagram

## Overview

Fastify is a highly performant and low overhead web framework for Node.js. This document provides a comprehensive architecture diagram and explanation of its core components.

## Architecture Diagram

```mermaid
graph TB
    subgraph "Core Components"
        FastifyInstance["Fastify Instance<br/>(fastify.js)"]
        Router["Router<br/>(find-my-way)"]
        Server["HTTP/HTTPS/HTTP2 Server<br/>(server.js)"]
        Avvio["Plugin System<br/>(avvio)"]
    end

    subgraph "Request Lifecycle"
        Request["Request Object<br/>(request.js)"]
        Reply["Reply Object<br/>(reply.js)"]
        Context["Route Context<br/>(context.js)"]
        HandleRequest["Request Handler<br/>(handleRequest.js)"]
    end

    subgraph "Hooks System"
        ApplicationHooks["Application Hooks<br/>- onRegister<br/>- onReady<br/>- onListen<br/>- onClose"]
        LifecycleHooks["Lifecycle Hooks<br/>- onRequest<br/>- preParsing<br/>- preValidation<br/>- preHandler<br/>- preSerialization<br/>- onSend<br/>- onResponse<br/>- onError"]
        HookRunner["Hook Runners<br/>(hooks.js)"]
    end

    subgraph "Validation & Serialization"
        SchemaController["Schema Controller<br/>(schema-controller.js)"]
        Validator["Validator<br/>(validation.js)"]
        Serializer["Serializer<br/>(fast-json-stringify)"]
        AjvCompiler["AJV Compiler<br/>(@fastify/ajv-compiler)"]
    end

    subgraph "Parser & Content"
        ContentTypeParser["Content Type Parser<br/>(contentTypeParser.js)"]
        BodyParser["Body Parsers<br/>- JSON<br/>- Text<br/>- Custom"]
    end

    subgraph "Plugin System"
        PluginManager["Plugin Manager<br/>(pluginUtils.js)"]
        Encapsulation["Encapsulation<br/>(pluginOverride.js)"]
        Decorators["Decorators<br/>(decorate.js)"]
    end

    subgraph "Error Handling"
        ErrorHandler["Error Handler<br/>(error-handler.js)"]
        FourOhFour["404 Handler<br/>(fourOhFour.js)"]
        ErrorSerializer["Error Serializer<br/>(error-serializer.js)"]
    end

    subgraph "Logging"
        LoggerFactory["Logger Factory<br/>(logger-factory.js)"]
        Pino["Pino Logger<br/>(pino)"]
    end

    %% Main Flow
    Client([Client]) --> Server
    Server --> Router
    Router --> Context
    Context --> HandleRequest
    HandleRequest --> Request
    HandleRequest --> Reply

    %% Plugin System
    FastifyInstance --> Avvio
    Avvio --> PluginManager
    PluginManager --> Encapsulation
    Encapsulation --> Decorators

    %% Hooks Flow
    HandleRequest --> HookRunner
    HookRunner --> LifecycleHooks
    FastifyInstance --> ApplicationHooks

    %% Validation Flow
    HandleRequest --> Validator
    Validator --> SchemaController
    SchemaController --> AjvCompiler

    %% Serialization Flow
    Reply --> Serializer
    Serializer --> SchemaController

    %% Parsing Flow
    HandleRequest --> ContentTypeParser
    ContentTypeParser --> BodyParser

    %% Error Flow
    HandleRequest --> ErrorHandler
    Router --> FourOhFour
    ErrorHandler --> ErrorSerializer

    %% Logging
    FastifyInstance --> LoggerFactory
    LoggerFactory --> Pino
    Request --> Pino
    Reply --> Pino

    style FastifyInstance fill:#f9f,stroke:#333,stroke-width:4px
    style HandleRequest fill:#bbf,stroke:#333,stroke-width:2px
    style Router fill:#bbf,stroke:#333,stroke-width:2px
    style Avvio fill:#bfb,stroke:#333,stroke-width:2px
```

## Component Descriptions

### Core Components

#### Fastify Instance (`fastify.js`)
- Main entry point and factory function
- Creates and configures the server instance
- Manages global settings and state
- Provides the public API for route registration, plugins, decorators, etc.
- Current version: 5.3.3

#### Router (`find-my-way`)
- High-performance HTTP router
- Supports parametric routes, wildcards, and regular expressions
- Handles route constraints and versioning
- Manages route lookup and dispatching

#### HTTP Server (`server.js`)
- Creates HTTP, HTTPS, or HTTP2 server instances
- Handles server lifecycle (listen, close)
- Manages multiple address bindings (IPv4/IPv6)
- Supports keep-alive, timeouts, and connection management

#### Plugin System (`avvio`)
- Manages asynchronous plugin loading
- Provides encapsulation and inheritance
- Handles plugin dependencies and boot order
- Supports plugin timeouts and error handling

### Request Lifecycle Components

#### Request Object (`request.js`)
- Wraps Node.js IncomingMessage
- Provides convenient accessors for headers, params, query, body
- Supports request decorators
- Manages request ID generation

#### Reply Object (`reply.js`)
- Wraps Node.js ServerResponse
- Handles response serialization
- Manages response headers and status codes
- Supports streaming and async responses
- Implements reply decorators

#### Route Context (`context.js`)
- Stores route-specific configuration
- Manages route-level hooks
- Contains schema validators and serializers
- Handles route-specific error handlers

#### Request Handler (`handleRequest.js`)
- Orchestrates the request lifecycle
- Runs hooks in the correct order
- Handles body parsing and validation
- Manages error propagation

### Hooks System

#### Application Hooks
- **onRegister**: Called when a plugin is registered
- **onReady**: Called when the server is ready
- **onListen**: Called when the server starts listening
- **onClose**: Called when the server is closing
- **preClose**: Called before the close sequence

#### Lifecycle Hooks (per request)
1. **onRequest**: First hook, can modify request
2. **preParsing**: Before body parsing
3. **preValidation**: Before schema validation
4. **preHandler**: Before route handler
5. **preSerialization**: Before response serialization
6. **onSend**: Before sending response
7. **onResponse**: After response sent
8. **onError**: On any error
9. **onTimeout**: On request timeout
10. **onRequestAbort**: On request abort

### Validation & Serialization

#### Schema Controller
- Manages JSON schemas
- Compiles validators and serializers
- Supports custom compilers
- Handles schema references

#### Validation
- Uses AJV for JSON schema validation
- Validates headers, params, querystring, and body
- Supports custom error messages
- Can attach validation errors to request

#### Serialization
- Uses fast-json-stringify for performance
- Serializes response based on schema
- Supports custom serializers
- Handles different response codes

### Content Parsing

#### Content Type Parser
- Manages body parsing strategies
- Built-in parsers for JSON and text
- Supports custom parsers
- Handles content-type negotiation
- Implements body size limits

### Plugin System Features

#### Encapsulation
- Provides isolated contexts for plugins
- Inherits from parent context
- Prevents plugin conflicts
- Supports scoped decorators

#### Decorators
- Extends core objects (fastify, request, reply)
- Type-safe in TypeScript
- Supports getters/setters
- Inherited through encapsulation

### Error Handling

#### Error Handler
- Centralized error processing
- Custom error handlers per route
- Error serialization
- Status code mapping

#### 404 Handler
- Customizable not-found responses
- Supports encapsulated 404 handlers
- Integrates with router

### Logging

#### Logger Factory
- Creates scoped loggers
- Integrates with Pino
- Supports custom serializers
- Child loggers for requests

## Request Flow

1. **Client Request** → HTTP Server receives request
2. **Routing** → Router finds matching route
3. **Context Creation** → Route context initialized
4. **Request/Reply Objects** → Created with context
5. **Hooks Execution**:
   - onRequest hooks
   - preParsing hooks
   - Body parsing (if needed)
   - preValidation hooks
   - Schema validation
   - preHandler hooks
6. **Route Handler** → User-defined handler executes
7. **Response Processing**:
   - preSerialization hooks
   - Response serialization
   - onSend hooks
8. **Send Response** → Reply sent to client
9. **Cleanup**:
   - onResponse hooks
   - Logger output

## Key Design Principles

1. **Performance First**: Optimized for low overhead and high throughput
2. **Schema-Based**: JSON Schema for validation and serialization
3. **Extensible**: Rich plugin ecosystem with encapsulation
4. **Developer Friendly**: Excellent TypeScript support and developer experience
5. **Standards Compliant**: Follows HTTP specifications closely

## Performance Optimizations

- **V8 Optimizations**: Code structured for V8 optimization
- **Schema Compilation**: Validation and serialization compiled at startup
- **Efficient Routing**: Radix tree-based router
- **Minimal Allocations**: Reuses objects where possible
- **Fast Path**: Optimized paths for common cases

## Security Features

- **Prototype Poisoning Protection**: Built-in protection
- **Content Type Validation**: Strict content-type checking
- **Request Size Limits**: Configurable body limits
- **Error Information Hiding**: Production-safe error responses