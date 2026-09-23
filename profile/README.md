# InfraMesh

**Distributed AI inference across heterogeneous compute resources.**

InfraMesh is an open-source platform for building and operating distributed AI inference infrastructure across heterogeneous **GPU and CPU nodes**.

It connects independent compute resources into a unified inference network and provides centralized management, intelligent request routing, and extensible node integrations.

## Architecture

InfraMesh consists of three primary components:

```text
                         InfraMesh
                             │
                          Console
                             │
                    ┌────────┴────────┐
                    │                 │
                 Router            Worker
                    │                 │
                    └────────┬────────┘
                             │
                         AI Runtime
```

### Console

The **Console** is the control plane of InfraMesh.

It manages:

* Organizations and teams
* Router and Worker nodes
* Node registration and configuration
* Inference requests
* Routing policies
* Node runtime state
* Service API keys
* Usage and execution information

Applications send inference requests through the Console, which coordinates the appropriate Router and Worker nodes.

### Router

The **Router** is responsible for selecting the appropriate Worker for an inference request.

Routing decisions can be based on strategies such as:

* Round Robin
* Least Latency
* Least Busy
* AI-based Routing

Routers can use Worker capabilities and runtime information such as available models, latency, utilization, queue state, and hardware resources when making routing decisions.

The Router coordinates inference execution but does not perform inference itself.

### Worker

The **Worker** connects InfraMesh to the actual AI runtime.

Workers can expose heterogeneous compute environments including:

* GPU servers
* CPU-only nodes
* Local development machines
* On-premise infrastructure
* Cloud compute instances

A Worker receives an inference request and executes it using its configured AI runtime or framework.

## Repositories

### `infra-console`

The InfraMesh control plane.

Provides organization, team, node, routing, authentication, and inference management.

### `infra-router`

Router implementation for selecting Workers and coordinating inference requests across the InfraMesh network.

### `infra-node`

Core SDK and integrations for connecting **Router** and **Worker** nodes to InfraMesh.

It provides common interfaces, protocol specifications, DTOs, runtime integrations, and example implementations required to build custom InfraMesh nodes.

## Routing

InfraMesh separates **request routing** from **inference execution**.

```text
Application
    │
    ▼
 Console
    │
    ▼
 Router
    │
    ├── Worker A ── GPU
    ├── Worker B ── GPU
    └── Worker C ── CPU
```

This architecture allows inference workloads to be distributed across machines with different hardware, models, and runtime environments.

## Heterogeneous Compute

InfraMesh is designed around the idea that AI infrastructure does not need to consist of identical machines.

A single InfraMesh network can contain Workers with different:

* GPU models
* VRAM capacities
* CPU resources
* AI models
* Runtime environments
* Performance characteristics

The routing layer can use this information to select an appropriate execution node for each request.

## Extensible by Design

InfraMesh separates the control plane, routing layer, node SDK, and AI runtime.

This makes it possible to integrate different AI frameworks and runtimes without coupling the entire platform to a specific inference engine.

```text
InfraMesh
    │
    ├── Console
    │
    ├── Router
    │
    └── Worker
           │
           ├── Custom Runtime
           ├── Spring AI
           ├── vLLM
           └── Other AI Runtimes
```

## Project Structure

```text
InfraMesh
├── infra-console    # Control plane
├── infra-router     # Routing engine
└── infra-node       # Router / Worker SDK and integrations
```

## License

License information is available in each repository.
