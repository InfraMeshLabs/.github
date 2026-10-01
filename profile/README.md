# InfraMesh

**Distributed AI inference across heterogeneous compute resources.**

InfraMesh is an open-source platform for connecting and orchestrating distributed **GPU and CPU resources** as a unified AI inference infrastructure.

It provides a centralized **control plane and inference orchestration layer**, a common **Node SDK and protocol contracts**, and reference implementations for building Router and Worker nodes.

InfraMesh enables AI workloads to run across **personal computers, GPU servers, on-premise infrastructure, and cloud environments**, while allowing both **private and public AI resources** to participate in the same inference network.

Applications interact with InfraMesh through a common entry point, while the infrastructure handles routing, Worker selection, node connectivity, and inference execution across heterogeneous environments.

## Why InfraMesh?

AI models are evolving rapidly.

At the same time, the open AI ecosystem continues to grow. More models can now be downloaded, customized, and executed directly on consumer GPUs, personal computers, workstations, and privately owned servers.

Running an AI model on a single machine has become increasingly accessible.

However, using AI across **multiple computers and environments as a single infrastructure** is still considerably more complicated.

A developer or organization may already have:

- Personal computers with available GPU or CPU resources
- Developer workstations capable of running local AI models
- Dedicated GPU servers
- On-premise infrastructure
- Private AI models that must remain inside controlled environments
- Open models running through runtimes such as Ollama or vLLM
- Public or commercial AI services for workloads that do not require private infrastructure

These resources can all provide AI inference independently, but they are usually operated as separate systems.

Applications must know where models are running, how to communicate with each runtime, which nodes are available, and which AI resources are appropriate for each request.

As infrastructure grows, applications can gradually become responsible for:

- Tracking where models are running
- Managing multiple inference endpoints
- Selecting available inference servers
- Handling unavailable or overloaded nodes
- Integrating different AI runtimes
- Monitoring GPU, CPU, memory, latency, and request load
- Distinguishing between private and public AI resources
- Updating integrations whenever infrastructure changes

InfraMesh moves these concerns into a shared infrastructure layer.

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

A single personal computer can participate as a Worker today.

Additional computers, GPU servers, on-premise machines, or cloud resources can be connected later without requiring applications to redesign how they access AI.

This allows AI infrastructure to grow gradually from the compute resources that are already available.

## Private and Public AI

Not every AI workload should be executed in the same environment.

Some requests may contain sensitive or private information and need to remain within locally controlled infrastructure.

Other workloads may benefit from public or commercial AI services.

InfraMesh is designed to allow both environments to coexist within the same infrastructure.

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

Routing policies determine which resources are eligible for a request.

Private workloads can be restricted to locally controlled or private infrastructure, while other workloads can use a broader set of available AI resources.

Applications do not need to manage these environments independently.

They communicate with InfraMesh, while the infrastructure determines how requests should be handled according to the configured routing policies.

## What InfraMesh Enables

### Use the hardware you already have

Existing personal computers, developer workstations, GPU servers, and CPU machines can participate in the inference network.

Building distributed AI infrastructure does not have to begin with a dedicated GPU cluster.

### Scale incrementally

InfraMesh can start with a single machine.

Additional Workers can be connected as more compute resources become available.

Applications can continue using the same InfraMesh entry point while the underlying infrastructure grows.

### Utilize idle compute resources

GPU and CPU resources that would otherwise remain unused can participate in AI inference workloads.

This makes it possible to distribute workloads across available machines instead of relying entirely on a single inference server.

### Run AI close to your data

Private AI models can run on personal machines, internal GPU servers, or on-premise infrastructure.

Requests that require private processing can be routed to appropriate resources instead of being sent to external AI services.

### Combine private and public AI

Local AI and external AI services do not need to exist as completely separate infrastructures.

InfraMesh provides a common layer for managing different types of AI resources while preserving routing constraints between them.

### Avoid coupling applications to infrastructure

Applications communicate with InfraMesh rather than individual Workers or AI runtimes.

Changing a Worker, model runtime, or physical machine does not require applications to understand the entire infrastructure topology.

### Support heterogeneous environments

Workers do not need to use identical hardware or software.

Different machines can provide different:

- GPUs and CPUs
- Models
- AI runtimes
- Performance characteristics
- Resource capacities
- Deployment environments

### Route using runtime information

Routing decisions can consider runtime information such as:

- Worker availability
- Supported models
- Current request load
- Queue state
- Average latency
- GPU utilization
- VRAM utilization
- Private or public execution requirements

### Avoid runtime lock-in

InfraMesh does not attempt to replace AI runtimes such as Ollama, vLLM, Spring AI integrations, or custom inference systems.

Instead, it provides the infrastructure layer that connects, manages, and coordinates them across distributed compute resources.

Developers can also implement their own Router and Worker nodes using the common `infra-node` SDK.

## Platform Overview

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
       Console Routing                  AI Router
   Round Robin / Latency / Busy        infra-router
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

Depending on the configured routing strategy, Worker selection can be performed directly by the Console or delegated to a Router.

The selected Worker executes the inference request using its configured AI runtime.

## Components

### infra-console

The **InfraMesh Console** is the control plane and inference orchestration layer of the platform.

It manages the resources and configuration required to operate an InfraMesh network, including:

- Organizations and teams
- Router and Worker nodes
- Node registration and connectivity
- Routing strategies
- Service API keys
- Inference requests
- Node runtime state
- Usage and execution information

Applications send inference requests through the Console.

The Console determines how the request should be routed according to the configured routing strategy.

Worker selection can be handled directly by built-in routing strategies or delegated to a Router.

The Console then coordinates request execution with the selected Worker.

```text
Application
     │
     ▼
  Console
     │
     ├── Organization / Team Management
     ├── Authentication / API Keys
     ├── Node Management
     ├── Routing
     └── Inference Orchestration
```

The Console acts as the primary entry point for applications using the InfraMesh inference network.

---

### infra-node

**InfraMesh Node** is the core SDK and common protocol foundation for building Router and Worker nodes.

It defines the contracts shared between the Console, Router, and Worker implementations.

It provides:

- Common request and response specifications
- Router contracts and DTOs
- Worker contracts and DTOs
- Node communication contracts
- Node health information
- Runtime integrations
- Shared infrastructure
- Integration components

```text
                     infra-node
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
           Router                 Worker
      Implementation         Implementation
```

Developers can use `infra-node` to build their own Router or Worker implementations without depending on the provided reference projects.

This keeps communication contracts consistent while allowing implementations to use different routing algorithms, AI runtimes, frameworks, and infrastructure.

---

### infra-router

**InfraMesh Router** is a reference routing implementation built using `infra-node`.

Its responsibility is focused on **Worker selection**.

The Console provides the Router with an inference request and the eligible Worker candidates.

The Router evaluates those candidates and returns the Worker that should execute the request.

```text
Console
   │
   │ RoutingRequest
   │ + Worker Candidates
   ▼
 Router
   │
   │ Worker Selection
   ▼
Console
```

Routing decisions can consider information such as:

- Available Workers
- Supported models
- Request load
- Queue state
- Average latency
- GPU utilization
- VRAM utilization
- Request routing requirements

Example routing approaches can include:

- Runtime-aware routing
- Rule-based routing
- AI-assisted routing
- Custom routing algorithms

The Router does **not** execute AI inference.

It also does not need to manage the transport or runtime used by the selected Worker.

Its role is to make a routing decision and return the selected Worker to the Console.

---

### infra-worker

**InfraMesh Worker** is a reference Worker implementation built using `infra-node`.

A Worker represents an execution node capable of processing AI inference requests.

It receives requests from the InfraMesh infrastructure and delegates execution to its configured AI runtime.

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

Worker implementations can represent different environments such as:

- Personal computers
- Developer workstations
- GPU servers
- CPU-only machines
- On-premise infrastructure
- Cloud compute instances
- External AI providers

Different AI runtimes and frameworks can be integrated behind the Worker implementation.

Examples include:

- Ollama
- vLLM
- Spring AI integrations
- Custom inference runtimes
- External AI providers

This allows machines with different hardware, models, runtimes, and deployment environments to participate in the same InfraMesh network.

## Routing Architecture

InfraMesh separates routing orchestration from Worker selection.

The Console always remains the central orchestration point.

```text
                         Request
                            │
                            ▼
                         Console
                            │
                            ▼
                    Routing Strategy
                            │
            ┌───────────────┴───────────────┐
            │                               │
            ▼                               ▼
     Built-in Routing                  AI_ROUTER
            │                               │
     Round Robin                        Router
     Least Latency                         │
     Least Busy                            │
            │                               │
            └───────────────┬───────────────┘
                            │
                            ▼
                     Selected Worker
                            │
                            ▼
                       Inference
```

Built-in routing strategies allow the Console to select a Worker without involving an external Router.

When an AI Router strategy is configured, the Console builds the eligible Worker candidate list and delegates the selection decision to the Router.

The Router returns the selected Worker, while the Console continues to orchestrate the actual inference request.

## Node Connectivity

InfraMesh nodes can participate in the network without requiring applications to communicate with individual inference runtimes directly.

The Console maintains the infrastructure-level relationship between applications and execution nodes.

```text
Application
     │
     ▼
  Console
     │
     ▼
   Worker
     │
     ▼
AI Runtime
```

Node connectivity is separated from application-facing inference APIs.

This allows Worker implementations and their underlying runtimes to change without requiring applications to understand how individual nodes are deployed or connected.

## Session-Aware Routing

Distributed inference can require more than simply selecting the least busy Worker for every request.

Requests belonging to the same conversation or session may benefit from continuing on the same Worker.

InfraMesh supports session-aware routing so that repeated requests associated with the same session can maintain Worker affinity when appropriate.

```text
Session A
   │
   ├── Request 1 ──► Worker 2
   ├── Request 2 ──► Worker 2
   └── Request 3 ──► Worker 2
```

Routing state can be maintained independently from the application so that applications do not need to track which Worker handled previous requests.

## Bring Your Own Node

InfraMesh does not require Router and Worker nodes to use only the provided reference implementations.

Developers can use `infra-node` to implement nodes that fit their own infrastructure and AI stack.

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

A custom Worker could represent anything from a personal computer running a local model to a dedicated GPU server or an integration with an external AI provider.

A custom Router can implement routing policies appropriate for a particular infrastructure or workload.

This allows InfraMesh to work with heterogeneous hardware, models, frameworks, and inference runtimes without coupling the platform to a single implementation.

## Repositories

| Repository | Description |
| --- | --- |
| `infra-console` | Control plane and inference orchestration layer for the InfraMesh distributed AI inference network |
| `infra-node` | Core SDK and common protocol contracts for building Router and Worker nodes in the InfraMesh distributed AI inference network |
| `infra-router` | Reference routing engine for intelligent Worker selection in the InfraMesh distributed AI inference network |
| `infra-worker` | Reference Worker implementation for executing AI inference in the InfraMesh distributed AI inference network |

## Design Philosophy

InfraMesh is designed around a few core principles.

### Bring your own compute

AI infrastructure should not require specialized infrastructure from the beginning.

Personal computers, workstations, GPU servers, and existing compute resources should be able to participate in the network.

### Distributed by default

AI inference workloads can be executed across multiple independent compute nodes.

Infrastructure can grow from a single machine into a distributed network as additional resources become available.

### Heterogeneous infrastructure

Workers can have different GPUs, CPUs, models, capacities, runtime environments, and deployment locations.

InfraMesh treats this diversity as part of the infrastructure rather than requiring every node to be identical.

### Private and public AI coexistence

Private, locally hosted AI and public AI services serve different purposes.

InfraMesh is designed so that both can participate in the same infrastructure while routing policies determine which resources are appropriate for each request.

### Decoupled applications and infrastructure

Applications should not need to know where every model is running or which physical machine should execute a request.

InfraMesh provides a common infrastructure layer between applications and inference resources.

### Decoupled routing and execution

Routing and inference execution are separate responsibilities.

The Console orchestrates requests, routing logic selects Workers, and Workers execute inference.

This separation allows each responsibility to evolve independently.

### Extensible nodes

Router and Worker implementations can be customized using the common `infra-node` SDK and protocol contracts.

Developers are not restricted to the provided reference implementations.

### Runtime agnostic

InfraMesh is not tied to a single AI runtime, framework, model provider, or hardware environment.

Existing and future AI runtimes can participate through Worker implementations without changing the overall architecture.

### Application simplicity

Applications interact with a stable InfraMesh entry point rather than managing individual inference servers.

Infrastructure can change, expand, or move without requiring applications to track the underlying topology.

## License

License information is available in each repository.
