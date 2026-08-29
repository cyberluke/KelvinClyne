# Kelvin Clyne

<p align="center">
  <img src="https://raw.githubusercontent.com/cyberluke/KelvinClyne/master/assets/k.webp" alt="Kelvin Clyne in VS Code: Context Reactor, PlanGraph workflows, sub-agent orchestration, and the mission composer" width="1200">
</p>

<p align="center">
  <strong>Turn intent into verified execution.</strong><br>
  PlanGraph missions, parallel sub-agents, context control, and ground-truth gates for serious engineering in VS Code.
</p>

> **Pre-release:** Kelvin Clyne is under active qualification. Interfaces, model drivers, and execution policies may evolve between pre-release builds.

Kelvin Clyne is not another chat panel attached to an editor. It is an **agentic operations layer** for compiling high-level engineering intent into inspectable execution graphs, dispatching specialized sub-agents, controlling tools and context, and requiring evidence before work is declared complete.

The extension is designed for the part of AI-assisted engineering where a plausible answer is insufficient: long-running repository changes, local inference, heterogeneous model backends, multimodal research, runtime qualification, adversarial testing, and workflows that must survive contact with real systems.

**One agent surface. Skills, workflows, sub-agents, tools, context routing, retrieval, artifacts, and runtime intelligence underneath.**

## The operating principle

Most coding assistants optimize the conversation. Kelvin Clyne optimizes the **execution system**.

A model may produce convincing language while using stale context, selecting the wrong tool, silently losing intermediate state, or accepting a successful process exit as proof of correctness. Kelvin treats these as systems-engineering failures rather than prompting problems.

Its execution model is built around four invariants:

1. **Intent must become an explicit plan.** Complex work is compiled into a dependency graph rather than decomposed by an opaque sentence splitter.
2. **Independent work should execute concurrently.** Sub-agents are scheduled by dependency topology, not by arbitrary serial order.
3. **Context is a controlled resource.** Tools, retrieved knowledge, repository evidence, and artifacts enter the model context only when their expected value exceeds their token and attention cost.
4. **Completion requires ground truth.** A green command, fluent answer, or internally consistent trace is not sufficient evidence that the result is correct.

## From intent to execution

```text
High-level intent
    -> Mission Compiler
    -> typed PlanGraph (DAG)
    -> dependency-aware scheduler
    -> parallel sub-agents + tools + retrieval
    -> evidence and artifacts
    -> ground-truth gates
    -> verified result
```

### Mission Compiler and PlanGraph

The Mission Compiler transforms a proof-of-concept description, bug report, architectural objective, or implementation brief into a **directed acyclic graph of executable work**. Nodes represent bounded operations; edges encode real data, resource, and verification dependencies.

This changes orchestration from "generate a list and execute it" into a scheduling problem:

- non-overlapping branches can start immediately;
- dependent branches remain blocked until their inputs exist;
- failures are isolated to the affected branch;
- intermediate products become addressable artifacts;
- execution can be inspected, resumed, re-planned, or selectively retried;
- synthesis occurs only after the required branch evidence has converged.

The graph is not decorative planning output. It is the runtime contract between human intent, model reasoning, tools, and verification.

### Workflow selection

Not every task deserves a swarm. Kelvin exposes workflow topologies for direct execution, iterative build-review loops, structured deliberation, same-persona parallelism, and broad parallel validation. The selected topology controls scheduling, coordination cost, review depth, and the amount of evidence required before synthesis.

Simple linear changes stay linear. Complex missions gain parallelism only where the dependency graph supports it.

## Architecture

| Subsystem | Mechanism | Engineering effect |
|---|---|---|
| **Mission Compiler** | Converts high-level intent into a typed PlanGraph DAG | Makes dependencies, concurrency, and completion criteria explicit |
| **Sub-agent scheduler** | Dispatches independent graph branches in parallel | Reduces wall-clock latency without inventing unsafe concurrency |
| **Native model drivers** | Uses model-native tokenization and control syntax where available | Preserves provider-specific capabilities that generic compatibility layers erase |
| **OpenAI-compatible gateway** | Provides a broad interoperability fallback | Keeps non-native and self-hosted endpoints usable without defining the architecture around the lowest common denominator |
| **Programmable Tool Calls (PTC)** | Allows supported models to express multi-tool programs instead of emitting one JSON call at a time | Collapses tool round trips and enables parallel tool operations within one model turn |
| **Dynamic tool lifecycle** | Equips tools on demand and disposes them after use | Prevents the entire tool catalog from permanently occupying context |
| **Context Reactor** | Accounts for persona, system policy, tools, skills, MCP, chat, RAG, files, images, and reserved output | Turns context allocation into an observable resource-management problem |
| **Artifact storage** | Persists intermediate agent products outside the active prompt | Preserves work across turns while reducing repeated token expenditure |
| **Idea Well retrieval** | Combines embedding retrieval with reranking for code, research, and long-form references | Improves evidence density and avoids spending the context window on weak candidates |
| **Ground-truth gates** | Compares outputs against independent or invariant-based evidence | Detects silent semantic failures that ordinary crash detection cannot see |

## Native drivers: capability before compatibility

OpenAI-compatible APIs are useful transport adapters, but they are a poor universal intermediate representation. They routinely flatten model-specific control grammars, tokenizer behavior, tool semantics, cache affordances, and long-horizon execution features into a least-common-denominator schema.

Kelvin therefore prefers a **native-first driver strategy**:

- use the model's native tokenizer and control protocol when supported;
- preserve native tool and reasoning semantics instead of translating them prematurely;
- expose caching, batching, structured control, and context behavior at the driver boundary;
- fall back to an OpenAI-compatible gateway when portability is more valuable than specialization.

For Qwen-family runtimes, this includes support for native control syntax and programmable tool execution rather than forcing every operation through sequential JSON tool calls. A supported model can construct a short program that schedules multiple independent tool operations in a single API turn; practical concurrency remains bounded by policy, runtime limits, and the safety profile of the tools involved.

The result is not "one more provider integration." It is a protocol architecture that allows local models to retain the capabilities for which they were trained.

### Qwen 3.8 execution path

The Qwen 3.8 path is the clearest example of the native-first design. Kelvin uses the model-native tokenizer and XML control grammar over the transport rather than treating an OpenAI-compatible gateway as the semantic authority. Tools are equipped for the active graph node and disposed after use instead of being permanently injected into every prompt.

With PTC enabled, the current execution policy can schedule **up to ten eligible tool operations in parallel within one API turn**. The bound is deliberate: it creates meaningful round-trip compression while preserving validation, resource limits, and a comprehensible failure domain. Actual concurrency is further constrained by the selected tools and runtime.

## Context is memory bandwidth, not a scrapbook

Large context windows do not eliminate context engineering. They merely make inefficient context loading more expensive and less obvious.

The **Context Reactor** treats the prompt as a finite working set. It measures and routes:

- system and persona policy;
- current conversation state;
- active skills and workflow instructions;
- tool and MCP schemas;
- selected source files and multimodal inputs;
- retrieved code and research evidence;
- artifact references and cached intermediate results;
- reserved output capacity.

Tools are equipped when a graph node needs them and removed when the operation is complete. Repository context is selected through indexing, retrieval, and reranking rather than indiscriminate file injection. Intermediate results are written to artifact storage so that agents can reference stable products instead of repeatedly reconstructing them in the live prompt.

This matters most for local inference, where every unnecessary token consumes memory, latency, cache capacity, and attention that could have been spent on the task itself.

## Idea Well: retrieval for code and unfamiliar domains

**Idea Well** is Kelvin's built-in retrieval layer for repository knowledge, technical research, and long-form engineering references. It is intended for work that extends beyond the model's training data: new libraries, source-only implementations, internal architectures, entire books, evolving APIs, and best-practice corpora.

The retrieval pipeline combines:

1. structural and semantic indexing;
2. embedding-based candidate retrieval;
3. reranking for query-specific relevance;
4. context-budget-aware packing;
5. provenance carried into the execution artifact.

Embedding recall finds plausible candidates. Reranking determines which candidates deserve scarce context. The distinction is critical: high recall without discrimination simply moves the context-overload problem downstream.

During development, this retrieval and context path has been exercised with the BottleCap AI Qwen 3.6 runtime at a 96k context configuration with Multi-Token Prediction enabled. The point of that qualification was not to demonstrate that a large window can hold more text; it was to verify that routing, artifact reuse, and reranking leave more of that window available for actual reasoning and code generation.

## Programmable Tool Calls

Traditional tool calling serializes an agent around repeated model round trips:

```text
model -> one JSON call -> tool -> model -> next JSON call -> tool -> model
```

With **Programmable Tool Calls**, a capable model can emit a bounded program that describes control flow and independent operations. Kelvin validates the program against tool policy, applies concurrency and resource limits, executes eligible operations in parallel, and returns structured results to the graph.

This is particularly useful for operations such as:

- reading several independent source files;
- querying multiple indexes or runtime endpoints;
- collecting orthogonal diagnostics;
- launching non-conflicting repository analyses;
- comparing several candidate implementations or model backends.

PTC is not unrestricted arbitrary code execution. The program is an orchestration representation evaluated within the configured execution policy. Filesystem scope, command approval, network access, concurrency, timeouts, and tool availability remain explicit boundaries.

## Ground-truth execution

Kelvin separates three conditions that ordinary agent traces often conflate:

| Condition | What it proves | What it does **not** prove |
|---|---|---|
| The command exited successfully | The process reached a successful exit path | The produced result is semantically correct |
| The output is plausible | The result conforms to an expected surface pattern | The result corresponds to reality |
| Independent evidence agrees | The result survives an external oracle, invariant, differential check, or reproducible measurement | Only the scoped claim that was actually checked |

### Field report: a silent OpenVINO GPU correctness failure

While qualifying Qwen3-VL-8B on an Intel Xe iGPU, Kelvin localized a failure in the OpenVINO GPU plugin that did not crash, warn, or produce non-finite values. The dynamic-shape graph completed normally and returned plausible tensors, but the result was numerically wrong.

The same model IR and inputs produced the following measurements against a CPU oracle:

| Configuration | Cosine similarity vs CPU | Mean absolute magnitude | Result |
|---|---:|---:|---|
| GPU, dynamic token dimension | **0.146181** | **1.5666** | Incorrect |
| GPU, identical IR reshaped static | 0.998810 | 0.1287 | Correct |
| CPU reference | 1.000000 | 0.1292 | Oracle |

Kelvin bisected 1,586 instrumentable operations, verified the direct inputs of the first divergent node, reduced the failure to a reproducible subgraph, ruled out several attractive but incorrect hypotheses, and produced a self-contained public reproducer. Intel independently reproduced the dynamic-versus-static divergence and escalated it to the OpenVINO development team.

- [OpenVINO issue #37573: GPU plugin miscompiles the Qwen3-VL vision merger with a dynamic token dimension](https://github.com/openvinotoolkit/openvino/issues/37573)
- [Intel's independent reproduction](https://github.com/openvinotoolkit/openvino/issues/37573#issuecomment-5430257458)
- [Self-contained reproducer](https://gist.github.com/cyberluke/65d3206c0c2bad2f419bfb1c81121187)

The lesson is foundational:

> **Successful execution is not proof of correct execution.**

Crash detection is easy. Ground truth is harder. Kelvin checks both.

## Local-first; cloud when it earns its place

Kelvin is designed around local control rather than mandatory remote execution. Use local LLMs, VLMs, embedding models, rerankers, and developer tools on the hardware you own. Add cloud models or remote accelerators when they provide a measurable advantage in capability, context, latency, throughput, or economics.

The runtime model accommodates heterogeneous backends, including:

- GGUF and llama.cpp-class runtimes;
- OpenVINO on CPU, Intel GPU, and XPU-class devices;
- CUDA inference on NVIDIA GPUs;
- local or remote embeddings and rerankers;
- OpenAI-compatible endpoints;
- model-native cloud APIs;
- custom gateways and experimental runtimes.

Kelvin Clyne provides the VS Code control plane. **Kelvin AI Workstation** is the companion runtime layer for hardware intelligence, model management, secure execution, local services, and workstation orchestration.

Cloud is optional. Evidence is not.

## Agentic Ops and RedTeam work

Kelvin is built for both construction and adversarial inspection:

- long-running implementation missions;
- dependency-aware sub-agent orchestration;
- tool-use and protocol experiments;
- model and agent red teaming;
- sandbox and permission-boundary analysis;
- runtime correctness qualification;
- reproducible bug isolation;
- latency, throughput, cache, context, and utilization studies;
- local inference across unusual hardware combinations;
- failures that look successful until measured.

Security research is treated as an evidence discipline. Findings should be scoped, reproducible, and connected to an observable mechanism. A dramatic narrative is not a substitute for a minimal reproducer.

## What Kelvin asks of an agent

A Kelvin mission should be able to answer:

- What exact claim was implemented or investigated?
- Which repository and runtime evidence informed the decision?
- What dependencies constrained parallel execution?
- Which tools were used, under what policy, and with what outputs?
- What artifact records the intermediate result?
- Which independent check distinguishes correctness from plausibility?
- What remains uncertain, and what observation would resolve it?

The intended end state is not autonomous activity. It is **autonomous activity with inspectable causality**.

## Installation

Kelvin Clyne will initially be distributed as a public pre-release through the Microsoft Visual Studio Marketplace.

From VS Code:

1. Open **Extensions**.
2. Search for **Kelvin Clyne** by publisher **cyberluke**.
3. Enable **Pre-Release Version**.

Or install from the command line:

```bash
code --install-extension cyberluke.kelvin-clyne
```

The Extensions view is the canonical way to opt into the pre-release channel.

- [Visual Studio Marketplace publisher](https://marketplace.visualstudio.com/publishers/cyberluke)
- [Source repository](https://github.com/cyberluke/KelvinClyne)

> The first public builds are pre-release software. Use repository-scoped permissions, review command and network policies, and keep critical work under version control.

## Project status

The public pre-release concentrates on the real vertical slice:

- one agent surface in VS Code;
- Mission Compiler and PlanGraph execution;
- workflow selection and parallel sub-agent scheduling;
- native-first and OpenAI-compatible model connectivity;
- programmable and conventional tool calls;
- context accounting and routing;
- artifact-backed agent memory;
- embedding retrieval plus reranking;
- local-first runtime integration;
- evidence-oriented execution and diagnostics.

Some drivers, workflows, and RedTeam capabilities are experimental by design. Experimental does not mean unmeasured: behavior should be observable, failure should be containable, and every serious claim should be reproducible.

## Operations room

The community lives at [r/KelvinClyne](https://www.reddit.com/r/KelvinClyne/). It is the operations room for AI coding agents, local inference, RedTeam research, agent orchestration, and experimental developer infrastructure.

Field reports are welcome:

- screenshots that reveal real execution state;
- benchmarks with enough methodology to reproduce them;
- broken agents and minimized failure cases;
- strange hardware that should not work but does;
- workflows that survived contact with a real repository;
- correctness failures hidden behind successful execution;
- patches, drivers, graph strategies, and retrieval experiments.

Areas of active interest include RedTeam, exploit research, Agentic Ops, Mission Graphs, local models, Kelvin AI Workstation, Runtime Intel, and systems that probably should not work yet but somehow do.

## Reporting bugs and research findings

A useful report contains:

- the smallest reproducible mission or graph;
- extension, model, runtime, driver, and operating-system versions;
- hardware and relevant driver versions;
- expected and observed behavior;
- logs or artifacts with secrets removed;
- a correctness oracle or invariant where possible;
- whether the failure reproduces from a clean process;
- whether static/dynamic shapes, quantization, caching, concurrency, or tool policy change the outcome.

For security-sensitive reports, do not publish weaponized details before a responsible disclosure path has been established. Open a minimal issue that requests a private channel without disclosing the exploit chain.

## Research ethos

Kelvin Clyne is built around a simple hierarchy of confidence:

```text
assertion < plausible output < successful execution < measurement < independent reproduction
```

The objective is not to make the agent sound certain. The objective is to make uncertainty visible, turn hypotheses into experiments, and drive the system toward independently reproducible evidence.

**Build it. Break it. Measure it. Make the agent prove it.**
