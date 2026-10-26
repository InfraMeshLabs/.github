# InfraMesh

**Distributed AI inference across heterogeneous compute resources.**

InfraMesh is an open-source platform for connecting and orchestrating distributed **GPU and CPU resources** as a unified AI inference infrastructure.

It provides a centralized **control plane and inference orchestration layer**, a common **Node SDK and protocol contracts**, and extensible Router and Worker implementations.

InfraMesh enables AI workloads to run across **personal computers, GPU servers, on-premise infrastructure, and cloud environments**, allowing both **private and public AI resources** to participate in the same inference network.

Applications interact with InfraMesh through a common entry point, while the infrastructure manages routing, Worker selection, node connectivity, and inference execution across heterogeneous environments.

## Why InfraMesh?

AI models are evolving rapidly, and the open AI ecosystem continues to grow.

More models can now be downloaded, customized, and executed directly on consumer GPUs, personal computers, workstations, and privately owned servers.

Running an AI model on a single machine has become increasingly accessible.

However, using AI across **multiple computers and environments as a single infrastructure** remains considerably more complicated.

A developer or organization may already have:

- Personal computers with available GPU or CPU resources
- Developer workstations capable of running local AI models
- Dedicated GPU servers
- On-premise infrastructure
- Private AI models that must remain inside controlled environments
- Open models running through Ollama or vLLM
- Public or commercial AI services

These resources can provide AI inference independently, but they are usually operated as separate systems.

Applications must know where models are running, how to communicate with each runtime, which nodes are available, and which resources are appropriate for each request.

As infrastructure grows, applications can gradually become responsible for:

- Tracking models and inference endpoints
- Selecting available inference servers
- Handling unavailable or overloaded nodes
- Integrating different AI runtimes
- Monitoring resource utilization and request load
- Enforcing private and public execution requirements
- Updating integrations whenever infrastructure changes

InfraMesh moves these responsibilities into a shared infrastructure layer.

### The Idea

InfraMesh is built around a simple idea:

> **AI should be able to run wherever compute resources are available, while applications interact with them as a single AI infrastructure.**

Instead of treating every computer and AI runtime as an isolated inference server, InfraMesh connects them into a shared inference network.

```text
                         Application
                              │
                              ▼
                          InfraMesh
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
         Personal PC      GPU Server      Cloud Node
              │               │               │
           Local AI       Private AI       Public AI
              │               │               │
            Ollama           vLLM          Provider
```

A single personal computer can participate as a Worker.

Additional computers, GPU servers, on-premise machines, or cloud resources can be connected later without requiring applications to redesign how they access AI.

This allows AI infrastructure to grow gradually from the compute resources already available.

## Key Features

### Distributed AI inference

Connect multiple independent compute resources into a unified inference network.

Workers can operate on personal computers, GPU servers, CPU-only machines, on-premise infrastructure, and cloud environments.

### Heterogeneous compute

Workers can use different:

- GPUs and CPUs
- AI models
- Inference runtimes
- Resource capacities
- Deployment environments

InfraMesh does not require identical hardware or software across nodes.

### Intelligent Worker routing

Select Workers using built-in routing strategies or custom Router implementations.

Routing decisions can consider Worker availability, supported models, request load, latency, resource utilization, and execution requirements.

### Private and public AI

Allow private infrastructure and external AI services to participate in the same network.

Routing policies determine which resources are eligible for each request.

### Outbound node connectivity

Connect Router and Worker nodes through persistent WebSocket connections initiated by the nodes themselves.

This allows nodes behind NAT or inside private networks to participate without exposing their own public inference endpoints.

### Session-aware routing

Maintain Worker affinity for requests belonging to the same logical session.

### Worker capacity management

Control concurrent inference execution through Worker capacity limits and in-flight request reservations.

### Runtime independence

Integrate Ollama, vLLM, Spring AI, custom inference runtimes, and external AI providers through Worker implementations.

### Extensible Router and Worker SDK

Build custom nodes using shared protocol contracts, authentication, health monitoring, outbound connectivity, and structured error handling.

### OpenAI-compatible inference

Integrate applications through supported OpenAI-compatible API conventions without requiring direct access to individual Workers.

## Architecture

InfraMesh separates **infrastructure management**, **routing decisions**, and **inference execution** into distinct responsibilities.

```text
                         Application
                              │
                              ▼
                    ┌─────────────────┐
                    │     Console     │
                    │  infra-console  │
                    └────────┬────────┘
                             │
                     Routing Strategy
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
       Built-in Routing                 AI Router
       ROUND_ROBIN                    infra-router
       LEAST_LATENCY                       │
       LEAST_BUSY                          │
              │                             │
              └──────────────┬──────────────┘
                             │
                       Worker Selection
                             │
                             ▼
                    ┌─────────────────┐
                    │     Worker      │
                    │  infra-worker   │
                    └────────┬────────┘
                             │
                             ▼
                         AI Runtime

          Router / Worker built with infra-node
```

The **Console** is the primary entry point into the InfraMesh network.

Depending on the configured routing strategy, Worker selection is performed directly by the Console or delegated to a Router.

The selected Worker executes the inference request using its configured AI runtime.

The Router does not execute inference or communicate directly with the selected Worker.

## Components

### infra-console

**InfraMesh Console** is the control plane and inference orchestration layer.

It manages:

- Organizations, Teams, and Team Members
- Router and Worker nodes
- Node registration and connectivity
- Routing strategies
- Team Member API Keys
- Node credentials
- Inference requests
- Session affinity
- Worker capacity
- Node runtime state
- Audit logs and operational information

Applications send inference requests to the Console.

The Console authenticates requests, determines eligible Workers, applies routing policies, and coordinates inference execution.

Worker selection can be performed through built-in strategies or delegated to a custom Router.

The Console remains responsible for orchestration regardless of the routing strategy.

### infra-node

**InfraMesh Node** is the core SDK and common protocol foundation for Router and Worker implementations.

It provides:

- Shared Router and Worker protocol contracts
- Inference and chat DTOs
- Node registration and connection contracts
- Node API Key authentication
- Node health monitoring
- CPU, memory, and GPU monitoring
- Outbound WebSocket connection infrastructure
- Automatic reconnection and heartbeat
- Streaming and cancellation support
- Active request tracking
- Structured error propagation
- Spring Boot integration

```text
                     infra-node
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
           Router                 Worker
      Implementation         Implementation
```

The SDK provides reusable infrastructure so that Router and Worker implementations can focus on their own business logic.

Developers can build custom nodes without depending on the provided reference implementations.

### infra-router

**InfraMesh Router** provides reference implementations for Worker selection.

The Console supplies the Router with an inference request and eligible Worker candidates.

The Router evaluates the request and candidates, then returns the Worker that should execute inference.

```text
Console
   │
   │ RoutingRequest
   │ + Worker Candidates
   ▼
 Router
   │
   │ RoutingResponse
   ▼
Console
```

Routing implementations can consider:

- Worker availability
- Supported models
- Request complexity
- Current request load
- Queue state
- Average latency
- GPU and VRAM utilization
- Routing constraints

Router implementations can use rule-based algorithms, runtime metrics, AI-assisted decisions, or custom policies.

The provided Laya-based Router demonstrates request-complexity-aware routing.

The Router does **not** execute inference or call Workers directly.

Custom Routers can also reject requests using structured error codes when their routing policies cannot be satisfied.

### infra-worker

**InfraMesh Worker** provides reference implementations for executing AI inference.

A Worker receives an inference request from the Console and delegates execution to its configured AI runtime.

```text
Console
   │
   │ Inference Request
   ▼
 Worker
   │
   ▼
AI Runtime
   │
   ▼
Inference Result
```

Worker implementations can represent:

- Personal computers
- Developer workstations
- GPU servers
- CPU-only machines
- On-premise infrastructure
- Cloud compute instances
- External AI providers

Supported integration approaches include:

- Ollama
- vLLM
- Spring AI
- Custom inference runtimes
- External AI services

Workers can support blocking and streaming inference depending on their implementation.

## Routing & Worker Selection

InfraMesh separates routing orchestration from Worker selection.

The Console supports four routing strategies.

| Strategy | Description |
| --- | --- |
| `ROUND_ROBIN` | Distributes requests across eligible Workers in rotation. |
| `LEAST_LATENCY` | Selects a Worker based on observed latency. |
| `LEAST_BUSY` | Selects a Worker based on current workload. |
| `AI_ROUTER` | Delegates Worker selection to a Router implementation. |

Built-in strategies select Workers directly within the Console.

With `AI_ROUTER`, the Console provides eligible Worker candidates to the Router and validates the returned selection.

### Worker eligibility

Before selecting a Worker, the Console evaluates applicable constraints, including:

- Node availability
- Enabled and drain states
- Streaming compatibility
- Private execution requirements
- Worker capacity

Custom Routers can apply additional selection policies using the candidate information and routing hints they receive.

Routing strategies determine which eligible Worker should execute a request.

They do not change the responsibility of the Console to orchestrate inference execution.

### Structured routing errors

Router implementations can return structured refusal codes when a request cannot satisfy their policies.

InfraMesh supports refusal codes such as:

| Error Code | Meaning |
| --- | --- |
| `PRIVATE_DATA_NO_LOCAL_WORKER` | No eligible local Worker satisfies the private execution requirement. |
| `SENSITIVE_DATA_NO_LOCAL_WORKER` | A Router-specific sensitive-data policy requires local execution, but no eligible local Worker is available. |

Supported refusal codes are propagated through the Console as HTTP `422` responses.

The Console returns standardized client-facing messages rather than exposing arbitrary Router error messages.

Unknown Router error codes are handled as upstream response errors.

## Node Connectivity

InfraMesh supports two connection modes for Router and Worker nodes.

### DIRECT

```text
Console ───── HTTP ─────▶ Router / Worker
```

In DIRECT mode, the Console sends HTTP requests to the configured Node endpoint.

The Node must be reachable from the Console.

DIRECT is the default connection mode.

### OUTBOUND

```text
Router / Worker
      │
      │ Persistent WebSocket
      ▼
   Console
```

In OUTBOUND mode, the Node initiates a persistent WebSocket connection to the Console.

This allows a Node to participate without exposing its own public IP address or inbound HTTP port.

OUTBOUND is particularly useful for:

- Personal computers
- Developer workstations
- Nodes behind NAT
- Private networks
- Environments where inbound connections are restricted

The Node must still be able to establish an outbound connection to the Console.

### Outbound Connection SDK

The `infra-node` SDK provides the common outbound connection infrastructure.

It handles:

- WebSocket connection lifecycle
- Node credential authentication
- Automatic reconnection
- Exponential backoff
- Heartbeat
- Runtime health reporting
- Request dispatch
- Streaming responses
- Request cancellation
- Graceful shutdown

Router and Worker implementations only need to provide their routing or inference logic.

Both DIRECT and OUTBOUND nodes participate in the same routing and inference architecture.

## Private & Public AI

Not every AI workload should be executed in the same environment.

Some requests require private processing, while others can use public or commercial AI services.

InfraMesh allows both environments to coexist.

```text
                         Application
                              │
                              ▼
                          InfraMesh
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
            Private AI                 Public AI
                 │                         │
        ┌────────┴────────┐         External / Cloud
        │                 │          AI Providers
        ▼                 ▼
   Personal PC       On-Premise
   / GPU Server      AI Server
```

### Explicit privacy constraints

Applications can specify private execution requirements through routing parameters such as `privateData`.

When private execution is required, the Console excludes Workers classified as external resources.

A local Worker represents a resource classified for private execution. It does not necessarily have to run on the same physical machine as the application.

If no eligible local Worker is available, the Console rejects the request rather than silently routing it to an external provider.

### Sensitive-data classification

Private execution requirements and automatic sensitive-data detection are different responsibilities.

InfraMesh enforces explicit routing constraints, but **does not automatically classify sensitive information by default**.

Custom Router implementations may provide additional classification or security policies.

Such classification depends on the Router implementation and should not be assumed to provide complete detection of sensitive information.

This separation allows InfraMesh to support different security requirements without coupling the platform to a specific classification model.

## Session Affinity & Capacity

### Session affinity

Requests belonging to the same conversation may benefit from continuing on the same Worker.

InfraMesh supports Redis-based session affinity.

```text
Session A
   │
   ├── Request 1 ──► Worker 2
   ├── Request 2 ──► Worker 2
   └── Request 3 ──► Worker 2
```

When session affinity is enabled, the Console attempts to reuse the previously selected Worker.

An existing affinity binding is validated against current Worker constraints before reuse.

If the binding is unavailable or invalid, normal routing is performed.

Session affinity can be configured independently for different Teams.

When an affinity binding is reused, the Console does not invoke the Router for that request.

Applications do not need to maintain Worker affinity themselves.

### Worker capacity

InfraMesh supports Worker concurrency limits through in-flight request reservations.

Capacity management helps prevent additional requests from being assigned to Workers that have reached their configured concurrency limits.

Reservation storage supports:

- In-memory tracking
- Redis-based shared tracking

Redis-based tracking allows multiple Console instances to coordinate Worker capacity.

Capacity management and session affinity are separate mechanisms.

Session affinity determines whether an existing Worker should be reused, while capacity management controls whether that Worker can accept additional requests.

## Security & Identity

InfraMesh separates application authentication, Node authentication, and Node connection credentials.

### Application authentication

The Console provides JWT-based authentication for management operations.

Organizations, Teams, and Team Members provide the administrative structure for managing infrastructure access.

### Team Member API Keys

Applications can authenticate inference requests using Team Member API Keys.

These keys associate requests with a specific Team Member and Team.

This allows inference requests to carry authenticated identity context through the infrastructure.

### Node authentication

DIRECT Node communication can use Node API Keys.

These credentials are separate from application-facing authentication.

### Outbound Node credentials

OUTBOUND nodes authenticate their WebSocket connections using Node credentials issued by the Console.

Node credentials support:

- Credential issuance
- Multiple credentials per Node
- Credential rotation
- Credential revocation
- Optional expiration
- Connection termination when credentials are revoked or expire

Credentials are used during connection authentication and are not included in inference message payloads.

### Identity propagation

The common protocol can propagate authenticated identity information, including:

- Team identity
- Team Member identity
- Session identity

These identifiers can help custom Router and Worker implementations implement scoped application features.

They are identifiers, not authentication credentials.

InfraMesh does not provide automatic long-term conversation memory. Custom implementations can use identity context to integrate their own memory systems.

### Audit logging

The Console records supported administrative and security-related operations through audit logs.

Audit retention helps manage accumulated operational records.

## OpenAI-Compatible API

InfraMesh provides OpenAI-compatible inference endpoints for applications that use supported OpenAI API conventions.

Applications communicate with the Console instead of individual inference servers.

```text
OpenAI-Compatible Client
          │
          ▼
     InfraMesh Console
          │
          ▼
     Routing Strategy
          │
          ▼
         Worker
          │
          ▼
       AI Runtime
```

This allows applications to use the InfraMesh inference network without managing individual Worker endpoints.

Supported operations include chat inference and streaming responses.

Compatibility applies to the API operations and features implemented by InfraMesh; it does not imply complete compatibility with every OpenAI API feature.

## Getting Started

InfraMesh can be deployed using the Console and one or more Worker nodes.

### 1. Start InfraMesh Console

Deploy the Console using a supported distribution method.

Available distribution approaches include:

- Docker
- Standalone packages for supported operating systems

### 2. Configure the infrastructure

Use the Console to create the required Organization and Team configuration.

Configure authentication and routing settings.

### 3. Register a Worker

Connect a Worker using DIRECT or OUTBOUND mode.

The Worker can run on a personal computer, GPU server, or another supported environment.

### 4. Configure routing

Select a routing strategy for the Team.

Built-in strategies can select Workers directly.

Alternatively, configure a Router to provide custom Worker selection logic.

### 5. Send an inference request

Applications send inference requests to the Console using a supported API.

```text
Application
     │
     ▼
  Console
     │
     ▼
Routing Strategy
     │
     ▼
   Worker
     │
     ▼
AI Runtime
```

The Console handles Worker selection and request orchestration.

Additional Workers can be connected later without changing the application's infrastructure integration.

Refer to the individual repositories for installation instructions, configuration, and API examples.

## Bring Your Own Node

InfraMesh does not require Router and Worker nodes to use only the provided reference implementations.

Developers can use `infra-node` to build nodes that fit their own infrastructure and AI stack.

```text
                         infra-node
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
    infra-router      Custom Router      Custom Worker
                                               │
                                    ┌──────────┼──────────┐
                                    ▼          ▼          ▼
                                Spring AI     vLLM      Custom
```

A custom Worker can integrate:

- Local AI models
- GPU inference servers
- Spring AI applications
- Custom inference engines
- External AI providers

A custom Router can implement:

- Rule-based routing
- Request-complexity-aware routing
- Model-aware routing
- Runtime-aware routing
- Custom policy enforcement

The SDK provides shared communication contracts and connection infrastructure while leaving routing decisions and inference execution to individual implementations.

This allows InfraMesh to support heterogeneous hardware, frameworks, and AI runtimes without coupling the platform to a single implementation.

## Repositories

| Repository | Description |
| --- | --- |
| `infra-console` | Control plane and inference orchestration layer for the InfraMesh distributed AI inference network |
| `infra-node` | Core SDK, shared protocol contracts, and common Node infrastructure |
| `infra-router` | Reference Router implementations for intelligent Worker selection |
| `infra-worker` | Reference Worker implementations for executing AI inference |

## Design Philosophy

InfraMesh follows several core principles.

### Bring your own compute

AI infrastructure should be able to use existing personal computers, workstations, GPU servers, and other available compute resources.

### Distributed and heterogeneous

Inference workloads can run across independent nodes with different hardware, models, runtimes, and capacities.

### Decoupled routing and execution

The Console orchestrates requests, Routers make selection decisions, and Workers execute inference.

Each responsibility can evolve independently.

### Private and public AI coexistence

Locally controlled infrastructure and external AI services can participate in the same network while respecting configured routing constraints.

### Runtime agnostic

InfraMesh does not replace AI runtimes.

It connects and coordinates existing inference systems through a common infrastructure layer.

### Extensible by design

Developers can implement custom Router and Worker nodes using shared SDK contracts without depending on reference implementations.

### Application simplicity

Applications communicate with a stable InfraMesh entry point rather than managing individual inference servers.

Infrastructure can change, expand, or move without requiring applications to understand the underlying topology.

## License

License information is available in each repository.
