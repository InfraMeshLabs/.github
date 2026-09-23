# InfraMesh

**Distributed AI inference across heterogeneous compute resources.**

InfraMesh is an open-source platform for connecting and utilizing distributed **GPU and CPU resources** for AI inference.

It provides a centralized control plane, a common Node SDK, and reference implementations for building Router and Worker nodes that participate in the InfraMesh network.

## Overview

InfraMesh separates the management of the inference network from the implementation of individual Router and Worker nodes.

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

The platform is organized around four repositories:

```text
InfraMesh
├── infra-console    # Control plane
├── infra-node       # Common Node SDK and specifications
├── infra-router     # Router reference implementation
└── infra-worker     # Worker reference implementation
```

`infra-router` and `infra-worker` are built on top of `infra-node` and provide example implementations that demonstrate how Router and Worker nodes can participate in the InfraMesh network.

## Components

### infra-console

The **InfraMesh Console** is the control plane of the platform.

It manages the resources and configuration required to operate an InfraMesh network, including:

* Organizations and teams
* Router and Worker nodes
* Node registration and configuration
* Routing strategies
* Service API keys
* Inference requests
* Node runtime state
* Usage and execution information

Applications send inference requests through the Console, which coordinates Router and Worker nodes according to the configured routing strategy.

---

### infra-node

**InfraMesh Node** is the core SDK for building nodes that participate in the InfraMesh network.

It defines the common contracts shared between the Console, Router, and Worker implementations.

It provides:

* Common request and response specifications
* Router interfaces and DTOs
* Worker interfaces and DTOs
* Node communication contracts
* Runtime integrations
* Shared infrastructure
* Example integration components

Developers can use `infra-node` to build their own Router or Worker implementations without depending on the provided reference projects.

```text
                     infra-node
                         │
              ┌──────────┴──────────┐
              │                     │
           Router                 Worker
        Implementation         Implementation
```

---

### infra-router

**InfraMesh Router** is a reference Router implementation built using `infra-node`.

It demonstrates how routing logic can be implemented to select an appropriate Worker for an inference request.

Routing strategies can consider information such as:

* Available Workers
* Supported models
* Request load
* Queue state
* Average latency
* GPU utilization
* VRAM utilization

Example routing strategies include:

* Round Robin
* Least Latency
* Least Busy
* AI-based Routing

The Router is responsible for routing decisions and coordination. It does not execute AI inference itself.

---

### infra-worker

**InfraMesh Worker** is a reference Worker implementation built using `infra-node`.

It demonstrates how a compute node can connect an AI runtime to the InfraMesh network.

A Worker receives inference requests and delegates execution to its configured AI runtime.

Worker implementations can represent different environments such as:

* GPU servers
* CPU-only machines
* Local development machines
* On-premise infrastructure
* Cloud compute instances

Different AI runtimes and frameworks can be integrated behind the Worker implementation.

## Architecture

The relationship between the projects can be represented as:

```text
                         Application
                              │
                              ▼
                       ┌─────────────┐
                       │   Console   │
                       │infra-console│
                       └──────┬──────┘
                              │
                  ┌───────────┴───────────┐
                  │                       │
                  ▼                       ▼
          ┌───────────────┐       ┌───────────────┐
          │    Router     │       │    Worker     │
          │ infra-router  │       │ infra-worker  │
          └───────┬───────┘       └───────┬───────┘
                  │                       │
                  │                       ▼
                  │                  AI Runtime
                  │
                  └──────────┐
                             │
                       Worker Selection

          ─────────────────────────────────────
                  Built with infra-node
          ─────────────────────────────────────
```

`infra-node` defines the common foundation, while `infra-router` and `infra-worker` demonstrate concrete implementations of those contracts.

## Bring Your Own Node

InfraMesh does not require Router and Worker nodes to use only the provided reference implementations.

Developers can use `infra-node` to implement nodes that fit their own infrastructure and AI stack.

```text
                         infra-node
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
    infra-router       Custom Router      Custom Worker
                                            │
                                  ┌─────────┼─────────┐
                                  ▼         ▼         ▼
                              Spring AI    vLLM     Custom
```

This allows InfraMesh to work with heterogeneous hardware, models, frameworks, and inference runtimes without coupling the platform to a single implementation.

## Repositories

| Repository      | Description                                                                               |
| --------------- | ----------------------------------------------------------------------------------------- |
| `infra-console` | Control plane for managing organizations, teams, nodes, routing, and inference requests   |
| `infra-node`    | Core SDK, common specifications, interfaces, and integrations for Router and Worker nodes |
| `infra-router`  | Reference Router implementation built with `infra-node`                                   |
| `infra-worker`  | Reference Worker implementation built with `infra-node`                                   |

## Design Philosophy

InfraMesh is designed around a few core principles:

**Distributed by default**
AI inference workloads can be executed across multiple independent compute nodes.

**Heterogeneous infrastructure**
Workers can have different GPUs, CPUs, models, capacities, and runtime environments.

**Decoupled routing and execution**
Routers decide where requests should run, while Workers are responsible for executing them.

**Extensible nodes**
Router and Worker implementations can be customized using the common `infra-node` SDK.

**Runtime agnostic**
InfraMesh is not tied to a single AI runtime or framework.

## License

License information is available in each repository.
