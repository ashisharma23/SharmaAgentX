Below is a consolidated **~400-word master prompt** that incorporates all the requirements you've built up: application-level PRs, configurable generation, application pattern discovery, Markdown context, knowledge graph/Cortex, context and memory governance, HITL microflows, observability, evaluations, hooks, Git, Podman, Kubernetes, and reference architectures.

Act as a **senior DevOps architect, software engineer, and agentic-AI platform specialist**. Design and implement an **implementation-ready, production-grade, configurable multi-agent SDLC platform/harness** for enterprise use. The solution must be deployable on **Kubernetes using Podman containers**, with independently deployable **Python backend and TypeScript frontend**.

### 1. Reference Architecture Research

Before implementation, research highly-rated, actively maintained GitHub repositories implementing similar **agentic AI, LangGraph, MCP, RAG, knowledge graphs, Kubernetes, observability, and agentic SDLC** solutions. Include repositories such as `langgraph-agent-stack`, `Multi-Agent-Orchestration`, and `agentic-development-lifecycle`, and identify additional relevant projects.

Create `docs/reference-architectures.md` containing:

**Repository | URL | Stars/Activity | Architecture Pattern | Applicability | Adopted/Modified/Rejected | License | Implementation Reference**

Do not blindly copy code. Respect licenses and document architectural decisions and deviations.

### 2. Technology Stack

Use:

* Python
* TypeScript
* LangGraph
* ArangoDB Knowledge Graph
* Snowflake Cortex
* MCP
* OpenTelemetry
* Git/GitHub-compatible workflow
* Podman
* Kubernetes/Helm

Use provider interfaces so LLM, MCP, retrieval, and knowledge-graph implementations can be replaced through configuration.

### 3. Epic and Application Model

The primary business input is an **Epic ID**. Retrieve and correlate:

* Business requirements
* Solution intent
* Features
* User stories
* Application scope
* Existing architecture
* TDD/design
* APIs
* Dependencies
* Integration patterns
* Data flows
* Related documentation and code

Treat **Application as a first-class execution, security, configuration, and PR boundary**.

An Epic may contain multiple applications. The user must be able to configure/select the application(s) in scope.

**PRs must always be created at application level.**

### 4. Application-Specific Configuration

Each application must be independently configurable and may share or use different:

* MCP servers/tools
* ArangoDB endpoint/database/graph
* Snowflake Cortex service/search
* LLM/model/provider
* Repository/branch
* Context policy
* Memory policy
* Guardrails
* Evaluation policy
* Deployment configuration

Support configuration inheritance:

**Global → Environment → Application → Workflow → Agent → Runtime Override**

Provide schema-driven YAML/JSON configuration and `.env.example`.

### 5. Configurable Generation

Allow users to independently enable/disable generation of:

* Features
* User Stories
* Acceptance Criteria
* Design/TDD
* Code
* Tests
* Documentation
* Pull Request

Generation must be configurable at **Epic and/or Application scope**.

Organizations must be able to customize the structure, fields, naming conventions, templates, and schemas of generated Features, Stories, Design documents, Code, Tests, and PR descriptions without modifying agent code.

### 6. Additional User Context

Allow users to upload Markdown (`.md`) files containing additional business or technical context at either:

* Epic scope
* Application scope

Maintain metadata including:

**scope, source, version, uploader, timestamp, application ID, epic ID, provenance, and validity/retention policy.**

Ensure this context is filtered and authorized before being passed to agents.

### 7. Application Pattern Discovery

Before generating design or code, the harness must inspect the application repository and enterprise knowledge sources to identify existing:

* Technology stack
* Languages/frameworks
* Repository structure
* Coding conventions
* Architectural patterns
* APIs
* Services
* Integration patterns
* Database/data-access patterns
* Messaging/event patterns
* Security/authentication
* Configuration patterns
* Logging/observability
* Testing patterns
* Deployment patterns
* Similar/related code artifacts

The system must **reuse and follow existing application patterns wherever appropriate**, rather than inventing new architecture or coding approaches.

Generate an explicit **Application Pattern Profile** that becomes controlled context for downstream agents.

### 8. Knowledge Graph and Context Retrieval

Use **ArangoDB** to maintain application-specific knowledge graphs containing entities such as:

**Application, Service, API, Epic, Feature, Story, Document, Database, Interface, Team, Dependency, Architecture Component, Test, Repository and Code Artifact.**

Use edge relationships such as:

**DEPENDS_ON, CALLS, IMPLEMENTS, USES, RELATED_TO, OWNS, INTEGRATES_WITH.**

Use a canonical schema with application-specific extensions.

Use **Snowflake Cortex** for enterprise context retrieval and ArangoDB for relationship traversal and contextual enrichment.

Every retrieved item must retain provenance and source references.

### 9. Context Management

Implement a dedicated **Context Management Layer**.

Never pass the entire workflow state to every agent. Each specialized agent must receive only the **minimum relevant, authorized, validated context** required for its task.

Support:

* Context filtering
* Relevance ranking
* Context compression
* Summarization
* Token budgets
* Provenance
* Citations
* Context lineage
* Context versioning
* PII/security filtering
* Context TTL
* Context retention
* Context policies per agent/application

Clearly classify information as:

**Retrieved Fact | Inference | Recommendation | Unknown/Missing**

### 10. Memory Management

Implement configurable short-term and long-term memory.

Memory must be configurable at:

**Global → Application → Workflow → Agent**

Support:

* Memory enabled/disabled
* Retention period
* Scope
* Storage backend
* Retrieval strategy
* Token limits
* Summarization
* Expiration
* Security classification

Prevent cross-application memory leakage.

### 11. Multi-Agent Architecture

Implement a **LangGraph supervisor/orchestrator** with specialized agents including:

1. Requirement Agent
2. Epic Context Agent
3. Knowledge Graph Agent
4. Application Pattern Agent
5. Solution Architecture Agent
6. TDD Agent
7. Application Impact Agent
8. Integration/API Agent
9. Feature/Story Agent
10. Development Agent
11. Test Agent
12. Code Review Agent
13. Documentation Agent
14. Security/Guardrail Agent
15. Evaluation Agent
16. PR Agent

Support:

**state management, parallel execution, conditional routing, retries, timeouts, checkpoints, human approval, failure recovery, and resumability.**

### 12. Human-in-the-Loop Microflows

Implement configurable **microflows** between agent handoffs.

Example:

**Agent A → Context Manager → Context Review → Agent B**

The UI must allow users to inspect:

* Context produced
* Context selected
* Context filtered out
* Source/provenance
* Context transformations
* Target agent
* Validation status

Users can:

**Approve | Edit | Reject | Retry | Continue**

Provide:

`HITL_ENABLED=true/false`

Support global, workflow, application, and agent-level overrides.

When disabled, the workflow executes automatically.

### 13. Application Design Package

At application scope, generate a version-controlled design package containing:

* Solution intent
* Architecture
* Application impact
* Component design
* Integration design
* API design
* Data flow
* Security considerations
* Deployment considerations
* Testing strategy
* Sequence diagrams
* Architecture/component diagrams

Use **Mermaid or PlantUML-compatible diagram definitions** so diagrams remain Git-versioned and editable.

### 14. Guardrails

Implement schema-driven guardrails for:

* Input/output validation
* Prompt injection
* Tool authorization
* MCP access
* Repository access
* Data leakage
* PII
* Secrets
* Unsafe code
* SQL/query safety
* Cross-application access
* Context leakage
* Hallucination/factuality
* PR authorization

Support:

**pre-agent, post-agent, pre-tool, post-tool, pre-context, post-context, pre-generation, post-generation, and pre-PR validation.**

### 15. Hooks

Provide configurable lifecycle hooks:

**pre/post workflow
pre/post agent
pre/post tool
pre/post retrieval
pre/post context
pre/post generation
pre/post evaluation
pre/post PR**

Hooks must support enterprise customization without changing core agent implementation.

### 16. Evaluation Framework

Implement reusable evaluation for:

* Retrieval accuracy
* Context relevance
* Context completeness
* Context leakage
* Factuality
* Citation correctness
* Hallucination
* Agent routing
* Tool selection
* Schema compliance
* Guardrail effectiveness
* Code quality
* Test quality
* Security
* Latency
* Token usage
* Cost
* HITL decisions
* End-to-end workflow correctness

Support golden datasets, regression tests, agent-level evaluations, workflow evaluations, and end-to-end evaluations.

### 17. Observability

Implement **OpenTelemetry-compatible telemetry, logging, metrics, and distributed tracing**.

Capture:

* Workflow traces
* Agent spans
* Tool/MCP calls
* Cortex retrieval
* ArangoDB queries
* Context transformations
* LLM metadata
* Token usage
* Latency
* Errors
* Retries
* Cost
* Guardrail results
* Evaluation results
* HITL decisions

Every event should support:

**application_id, epic_id, workflow_id, agent_id, context_id, trace_id, environment.**

Do not expose secrets or sensitive payloads in telemetry.

### 18. Git and PR Lifecycle

Generated artifacts must be Git-compatible and traceable.

Maintain:

**Epic → Application → Source → Retrieved Context → Context Version → Agent → Specification Version → Generated Artifact → Evaluation → Human Approval → Commit → PR**

PR creation must resolve the configured **application repository, branch strategy, reviewers, and PR policy**.

### 19. Production Deployment

Provide:

* Podman Containerfiles
* Kubernetes manifests
* Helm charts
* ConfigMaps
* Secrets
* RBAC
* Service Accounts
* Network Policies
* Health probes
* Resource limits
* HPA
* Ingress
* Observability
* CI/CD
* Security scanning

Frontend, backend, and agent services must be independently deployable.

### 20. Working Reference Implementation

Provide a fully runnable local example using:

`EPIC-12345`

Create at least **two applications with different MCP, ArangoDB, Cortex, repository, and context configurations**.

Provide mocked Cortex, ArangoDB, MCP, and LLM services so the complete workflow works without production credentials.

### 21. Repository Deliverables

Generate:

```text
/agents
/skills
/workflows
/specs
/schemas
/context
/memory
/guardrails
/evals
/hooks
/tools
/mcp
/backend
/frontend
/deploy
/k8s
/helm
/tests
/docs
/examples
/scripts
```

Include:

`README.md`
`Agents.md`
`Skills.md`
`Workflows.md`
`Context.md`
`Memory.md`
`Guardrails.md`
`Evaluation.md`
`Observability.md`
`Configuration.md`
`Contributing.md`
`docs/reference-architectures.md`

### 22. Validation Gate

Do not provide documentation alone. **Build and execute the reference implementation.**

Validate:

* Podman builds
* Local end-to-end workflow
* Agent execution
* Context filtering
* Memory policies
* HITL microflows
* Guardrails
* Hooks
* Evaluations
* Telemetry/tracing
* Frontend/backend integration
* Application isolation
* PR generation
* Git integration
* Kubernetes/Helm validation
* Security scanning
* Automated tests

The final solution must be **modular, configurable, observable, auditable, secure, application-aware, and extensible**, with enterprise behavior controlled through specifications and configuration rather than hard-coded agent logic.
