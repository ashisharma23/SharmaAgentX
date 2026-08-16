# The Enterprise Agentic AI Tool Stack
### A practitioner's guide for IT teams building agentic solutions in Telecom, Healthcare, and Finance

> Curated for Solution Architects, AI/ML Engineers, Platform Engineers, and Governance/Security leads who are designing, building, and operating production-grade agentic AI systems.

---

## How to use this guide

Agentic AI systems are not "one tool." A production system is a **stack**: a model layer, an orchestration layer, a memory/knowledge layer, an integration layer, a governance/security layer, and an operations layer sitting on top of an enterprise cloud platform. This document is organized so you can:

1. Understand the **layers** of the stack and what job each layer does.
2. Pick tools **by task category** (build, connect, remember, observe, evaluate, secure, govern, ship).
3. Pick tools **by persona** — what a Solution Architect, an Agent Engineer, or a Compliance Lead actually touches day to day.
4. Apply a **domain lens** for Telecom, Healthcare, and Finance, where regulatory and data-residency constraints shape tool selection more than raw capability does.

No tool here is a universal winner. Selection should be driven by: existing cloud commitment, regulatory exposure, team skill set, data residency requirements, and total cost of ownership — not benchmark leaderboards alone.

---

## Reference architecture — the agentic stack at a glance

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. GOVERNANCE & COMPLIANCE LAYER                                    │
│     NIST AI RMF · ISO/IEC 42001 · EU AI Act · Domain regs (HIPAA,   │
│     PCI-DSS, TM Forum) — policy, audit, human-approval workflows     │
├─────────────────────────────────────────────────────────────────────┤
│  2. SECURITY & GUARDRAILS LAYER                                      │
│     Prompt-injection defense · PII/output filtering · red-teaming    │
├─────────────────────────────────────────────────────────────────────┤
│  3. OBSERVABILITY & EVALUATION LAYER                                 │
│     Tracing · cost/latency monitoring · offline & online evals       │
├─────────────────────────────────────────────────────────────────────┤
│  4. ORCHESTRATION LAYER (agent framework / runtime)                  │
│     Planning, tool-calling, multi-agent coordination, state          │
├─────────────────────────────────────────────────────────────────────┤
│  5. INTEGRATION LAYER (protocols)                                    │
│     MCP (agent↔tool) · A2A (agent↔agent) · enterprise APIs           │
├─────────────────────────────────────────────────────────────────────┤
│  6. GOVERNED SEMANTIC LAYER  ← the layer most stacks skip            │
│     One definition of "revenue," "churn," "claim status" — served    │
│     to agents, BI, and humans alike, with access control at query    │
│     time and lineage back to the source                              │
├─────────────────────────────────────────────────────────────────────┤
│  7. MEMORY & KNOWLEDGE LAYER                                         │
│     Vector DB · knowledge graph · long-term/episodic memory          │
├─────────────────────────────────────────────────────────────────────┤
│  8. MODEL ACCESS LAYER (LLM gateway)                                 │
│     Routing, fallback, caching, budgets across model providers       │
├─────────────────────────────────────────────────────────────────────┤
│  9. FOUNDATION — enterprise cloud / agent platform                   │
│     Azure AI Foundry · AWS Bedrock AgentCore · Vertex AI · watsonx   │
└─────────────────────────────────────────────────────────────────────┘
```

**Rule of thumb:** the further down the stack, the more it is a *platform decision* (made once, cloud-wide); the further up, the more it is a *governance decision* (made once, org-wide, and enforced on every agent regardless of who built it). The orchestration and memory layers are where most day-to-day engineering choice happens — but the semantic layer between memory and the model access layer is where most production incidents actually originate. An agent can orchestrate flawlessly and still hand a customer the wrong number because "active policy" or "churned customer" was defined differently in two source systems.

---

## 1. Agent Orchestration Frameworks

The layer that turns an LLM into an agent: planning, tool invocation, state, and multi-agent coordination.

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **LangGraph** | Low-level, graph-based orchestration for stateful, long-running, cyclical agent workflows; explicit control over execution paths | Teams that need fine-grained control and complex branching logic; the safest default for new production builds | https://www.langchain.com/langgraph |
| **CrewAI** | Role-based "crews" (agents with roles/goals) + "flows" for deterministic process control | Fastest path to a working multi-agent prototype; readable, business-analyst-friendly agent design | https://www.crewai.com/ |
| **Microsoft Agent Framework** | Unified successor to AutoGen + Semantic Kernel; conversational multi-agent patterns + enterprise state/telemetry/middleware + graph workflows; native MCP & A2A support | Microsoft/.NET/Azure-native enterprises; first-class C# and Python parity | https://learn.microsoft.com/agent-framework |
| **AutoGen (AG2)** | Conversation-driven multi-agent orchestration; agents "talk" to solve tasks | Research-style multi-agent collaboration; still widely used, now folded into Microsoft Agent Framework | https://microsoft.github.io/autogen/ |
| **Semantic Kernel** | Lightweight orchestration + plugin/skill SDK, strong enterprise (.NET) integration | Enterprises standardizing on Microsoft stack needing planners/skills over full agent autonomy | https://learn.microsoft.com/semantic-kernel/ |
| **LlamaIndex (Agents)** | Data-framework-first agents, deep RAG/indexing integration | RAG-heavy agents where retrieval quality is the primary engineering problem | https://www.llamaindex.ai/ |
| **OpenAI Agents SDK** | Lightweight, provider-native agent primitives (handoffs, guardrails, tracing) | Teams standardized on OpenAI models wanting minimal abstraction | https://openai.github.io/openai-agents-python/ |
| **Google Agent Development Kit (ADK)** | Google's agent-building SDK with native A2A and MCP support | Gemini-first teams, GCP-native deployments | https://google.github.io/adk-docs/ |
| **AWS Strands Agents** | Model-driven agent SDK underpinning Bedrock AgentCore | AWS-native teams wanting a lightweight, production-oriented SDK | https://strandsagents.com/ |
| **Pydantic AI** | Type-safe, Python-native agent framework built on Pydantic validation | Python teams wanting lightweight, strongly-typed agents without heavy abstraction | https://ai.pydantic.dev/ |
| **Claude Agent SDK** | Anthropic's SDK for building agents on Claude, with native tool use, memory, and computer-use primitives | Teams building on Claude wanting first-party agent scaffolding | https://docs.claude.com |

**Architectural note:** frameworks give you agent *logic*, not production guarantees. None natively solve durable execution across failures, pre-dispatch approval gates, or cross-framework governance — those come from the durable-execution and governance layers below (Temporal, agent control planes, gateways).

---

## 2. Enterprise Agent Platforms (Hyperscaler / Full-Stack)

Managed, cloud-native platforms bundling model access, orchestration, identity, and observability into one control plane.

| Platform | Specialization | Best fit | Reference |
|---|---|---|---|
| **Azure AI Foundry (Agent Service)** | Full-stack agent operating system: identity (Entra Agent ID), orchestration, model router (1,700+ model catalog), M365/SharePoint data connectors, Agent365 governance control plane | Microsoft-native orgs; agents grounded in Office/SharePoint data; strongest for regulated orgs already on Azure compliance | https://azure.microsoft.com/en-us/products/ai-foundry |
| **AWS Bedrock AgentCore** | Framework-agnostic agent runtime, vault-backed identity/token management, tight S3/Lambda/Redshift integration | AWS-native engineering cultures; multi-framework teams; strong zero-trust/multi-tenant posture; deep FedRAMP/HIPAA documentation | https://aws.amazon.com/bedrock/agentcore/ |
| **Google Vertex AI Agent Builder** | Unified ML + agent stack, native A2A protocol support, deep BigQuery/Workspace integration | Gemini-first teams, ML-training-heavy workloads, GCP-native data estates | https://cloud.google.com/products/agent-builder |
| **IBM watsonx** | Purpose-built for regulated industries; 700+ connectors, EU AI Act compliance tooling, IP indemnification | Regulated industries (banking, healthcare) prioritizing compliance tooling and auditability out of the box | https://www.ibm.com/watsonx |
| **Salesforce Agentforce** | CRM-native autonomous workflows, Atlas Reasoning Engine, Einstein Trust Layer | Sales/service teams already on Salesforce | https://www.salesforce.com/agentforce/ |
| **ServiceNow AI Platform** | IT/HR operations agents, AI Control Tower for cross-department agent governance | ITSM/HR-process automation on ServiceNow | https://www.servicenow.com/ai/ |

**Decision heuristic:** choose the hyperscaler that matches your existing cloud/data commitment first; use benchmark comparisons only to break ties. Re-platforming an agent estate later is expensive.

---

## 3. LLM Gateways / Model Access Layer

Middleware between agents and model providers: routing, fallback, caching, cost control, and a single policy chokepoint.

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **LiteLLM** | Open-source, self-hosted, OpenAI-compatible proxy to 100+ model providers; virtual-key budgeting | Teams wanting full ownership/self-hosting and multi-provider routing without vendor lock-in | https://www.litellm.ai/ |
| **Portkey** | Managed gateway: routing across 200+ providers, semantic caching, guardrails, fallback chains, cost budgets | Teams wanting governance/guardrails out of the box without operating infrastructure | https://portkey.ai/ |
| **Kong AI Gateway** | AI-specific plugins on Kong's mature API gateway platform (rate limiting, OIDC SSO, request/response transforms) | Enterprises already standardized on Kong for API management | https://konghq.com/products/kong-ai-gateway |
| **Helicone** | Drop-in proxy, minimal-code observability + cost tracking | Fast setup where latency and simplicity outweigh deep enterprise governance | https://www.helicone.ai/ |
| **Cloudflare AI Gateway** | Managed, edge-native gateway integrated with the Cloudflare stack | Teams already running on Cloudflare wanting zero-ops routing | https://developers.cloudflare.com/ai-gateway/ |

**Why this layer matters architecturally:** the gateway is the natural place to enforce per-team budgets, model fallback (reliability), and a single audit trail — answering "who called what model, when, and at what cost" across every agent in the estate.

---

## 4. Integration & Interoperability Protocols

Standards that let agents talk to tools and to each other without bespoke point-to-point integration.

| Protocol | Specialization | Best fit | Reference |
|---|---|---|---|
| **Model Context Protocol (MCP)** | Open standard (JSON-RPC) connecting agents to external tools, data, and prompts; the dominant agent-to-*tool* layer, adopted by Anthropic, OpenAI, Google, Microsoft | Any agent needing standardized, reusable tool/data connectors instead of custom integrations | https://modelcontextprotocol.io/ |
| **Agent2Agent (A2A)** | Peer-to-peer protocol for cross-vendor, cross-framework agent collaboration; Agent Cards for capability discovery, task lifecycle management | Multi-agent systems spanning organizational or framework boundaries (e.g., procurement agent ↔ supplier's agent) | https://a2a-protocol.org/ |
| **Agent Communication Protocol (ACP)** | Lightweight, REST-native agent messaging (IBM BeeAI), no SDK required | Simpler, framework-agnostic agent messaging needs | https://agentcommunicationprotocol.dev/ |

**Decision rule:** reach for MCP by default for tool/data access. Add A2A only when work crosses an ownership boundary you don't control (a different team, vendor, or company) and needs asynchronous, stateful handoffs.

---

## 5. Memory & Knowledge Layer

### Vector databases (retrieval / RAG backbone)

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **Pinecone** | Fully managed, serverless, zero-ops; built-in inference (embeddings + reranking), hybrid search | Teams wanting zero infrastructure management at any scale | https://www.pinecone.io/ |
| **Qdrant** | Rust-based, fastest open-source option; strong metadata filtering, native sparse/ColBERT multi-vector support | Performance-sensitive, filter-heavy workloads; strongest free/open-source tier | https://qdrant.tech/ |
| **Weaviate** | Native hybrid search (vector + BM25 + filters), built-in vectorization modules | Teams wanting hybrid search with minimal custom pipeline work | https://weaviate.io/ |
| **Milvus / Zilliz Cloud** | Distributed architecture built for billion-scale vector search | Highest-throughput, largest-scale enterprise deployments | https://milvus.io/ |
| **pgvector** | Postgres extension adding vector search to a relational database | Teams that want embeddings, documents, and metadata in one SQL-queryable system (avoids a second database) | https://github.com/pgvector/pgvector |
| **Chroma** | Developer-friendly, lightweight | Prototyping and MVPs before a production data-platform decision is made | https://www.trychroma.com/ |

### Agent memory frameworks (persistence across sessions)

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **Mem0** | Fast, product-ready memory API (vector + graph + key-value), automatic memory extraction | General-purpose personalization; largest community, easiest drop-in | https://mem0.ai/ |
| **Zep / Graphiti** | Temporal knowledge-graph memory; tracks how facts change over time with provenance | Enterprise use cases needing "what did we know, and when" — strong for compliance-sensitive institutional memory | https://www.getzep.com/ |
| **Letta (formerly MemGPT)** | Tiered, self-editing memory architecture; memory as core agent state | Long-running, complex agents that need to manage their own context window | https://www.letta.com/ |
| **LangMem** | Memory module native to the LangChain/LangGraph ecosystem | Teams already standardized on LangGraph | https://langchain-ai.github.io/langmem/ |

**Architectural note:** memory and RAG are related but distinct layers — RAG retrieves *organizational knowledge*; agent memory persists *what happened in this agent's interactions*. Regulated domains (healthcare, finance) should treat memory stores as systems of record subject to the same retention, access-control, and audit requirements as any other PII-bearing datastore.

---

## 5a. The Governed Semantic Layer — the piece most stacks skip

> *Community input: this section was added following practitioner feedback from finance and healthcare deployments — thank you to Ashish for flagging it.*

Orchestration, memory, and observability get most of the attention because they're where agents visibly *fail*: a bad tool call, a lost session, a runaway loop. But a large share of production incidents are quieter than that — the agent completes the task, gives a confident answer, and the answer is **wrong because the underlying metric was never governed**.

Text-to-SQL and raw retrieval let an agent reason freely over a warehouse or lake, which means it can also compute "active customer," "churn," or "claim status" a different way than the dashboard your compliance team already signed off on. Without a governed semantic layer sitting between the agent and the data, orchestration quality is capped by data-definition quality — the best planner in the world still breaks on a metric it was never told the rules for.

A **governed semantic layer** solves this by defining metrics, dimensions, and access rules **once**, upstream of any consumer, and serving that single definition to BI tools, embedded apps, and AI agents alike — typically over SQL, REST, GraphQL, and increasingly MCP. This is what turns "the agent gave an answer" into "the agent gave the answer, grounded in a definition we can point to, with lineage and row-level access control enforced at query time" — the difference between a demo and something audit-ready.

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **Cube (Cube Core)** | Open-source (Apache 2.0) semantic layer with governed metrics served over SQL, REST, GraphQL, and a native MCP server; compile-time row-level security and pre-aggregation caching | Teams needing one decoupled layer to serve BI, embedded analytics, and AI agents consistently, including multi-warehouse environments | https://cube.dev/ |
| **dbt Semantic Layer (MetricFlow)** | Metric definitions live inside the dbt project, served through dbt's Semantic Layer APIs | Teams already centered on dbt for transformation who want metrics defined close to the models | https://www.getdbt.com/product/semantic-layer |
| **AtScale** | Enterprise semantic layer speaking MDX/DAX; strong for OLAP, Excel, and Power BI-heavy estates | Large regulated enterprises needing semantic governance across many BI tools, teams, and AI systems simultaneously | https://www.atscale.com/ |
| **Looker / LookML** | BI-native semantic modeling layer, tightly coupled to Looker and GCP | Organizations standardized on Looker/Google Cloud as the analytics platform of record | https://cloud.google.com/looker |
| **Snowflake Semantic Views / Databricks Metric Views** | Warehouse-native semantic definitions, no separate service to operate | Single-platform shops wanting the least additional infrastructure | https://docs.snowflake.com/ · https://docs.databricks.com/ |
| **Atlan (context layer)** | Wraps semantic layers with lineage, ownership, business glossary, and access policy, then exposes governed context to agents via MCP | Enterprises that already have a data catalog and want agent-facing governance layered on top rather than rebuilt | https://atlan.com/ |

**Architectural placement:** the semantic layer sits between the memory/knowledge layer and the model access layer — the agent (or its retriever) queries governed metrics through this layer rather than running ungoverned SQL directly against the warehouse. Treat it as mandatory, not optional, for any agent that answers on behalf of other people: a customer, a clinician, an auditor, or a regulator. For a single analyst querying their own sandbox, raw retrieval is fine. The moment the answer leaves that sandbox, the semantic layer stops being optional.

**Why this matters most in finance and healthcare:** both domains already require a documented, auditable definition of core metrics (regulatory capital, claim status, adverse-event rate) independent of any single application. A governed semantic layer is the natural place to encode that definition once and prove — to an auditor, not just to a code reviewer — that every agent, dashboard, and report drew from the same source of truth. Pair this layer with the audit-log and human-approval requirements in the [Governance, Risk & Compliance](#10-governance-risk--compliance) section below; the semantic layer supplies the "what was the ground truth" half of the audit trail, and the governance layer supplies the "who approved what happened next" half.

---

## 6. Observability, Tracing & Cost Monitoring

The layer that answers "what did the agent actually do, and what did it cost?" — non-negotiable before any production deployment.

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **LangSmith** | Framework-agnostic tracing, evaluation, prompt versioning; deepest integration with LangChain/LangGraph | LangChain/LangGraph shops; near-zero setup for first production deployment | https://www.langchain.com/langsmith |
| **Langfuse** | MIT-licensed, fully self-hostable tracing + evaluation + prompt management | Teams prioritizing data sovereignty, self-hosting, or multi-framework flexibility | https://langfuse.com/ |
| **Arize Phoenix** | ML-grade evaluation rigor, native RAGAS support, drift detection, embeddings analysis | RAG-heavy systems where retrieval-quality evaluation depth matters most | https://phoenix.arize.com/ |
| **Datadog LLM Observability** | Bolts LLM tracing onto existing APM, correlating AI signals with infra metrics | Enterprises already standardized on Datadog for infrastructure monitoring | https://www.datadoghq.com/product/llm-observability/ |
| **Honeycomb** | Event-based, deep distributed tracing | Teams needing very granular, high-cardinality trace analysis | https://www.honeycomb.io/ |

---

## 7. Evaluation & Testing

Moving from "looks good in a demo" to a repeatable, CI-gated quality bar. Agent evaluation is **trajectory-first** — you must score the path (tool calls, reasoning steps), not just the final answer.

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **Ragas** | RAG-pipeline evaluation: faithfulness, context precision/recall, hallucination detection | RAG-specific evaluation | https://github.com/explodinggradients/ragas |
| **DeepEval** | Pytest-native, Python-first framework; 50+ built-in metrics; strong agentic-trace support | Python teams wanting evaluation as unit tests inside existing CI | https://deepeval.com/ |
| **Promptfoo** | YAML-first, language-agnostic CLI; strong red-teaming (50+ vulnerability scans: injection, jailbreak, PII, excessive agency) | Cross-language teams; security/adversarial testing alongside correctness | https://www.promptfoo.dev/ |
| **Inspect AI** | Evaluation framework from the UK AI Security Institute | Public-sector and safety-focused evaluation workflows | https://inspect.aisi.org.uk/ |
| **OpenAI Evals** | Registry-style evaluation framework and open eval-set format | Benchmarking against community/registry eval sets | https://github.com/openai/evals |
| **MLflow (LLM evaluation)** | Evaluation integrated with existing experiment tracking; prompt auto-optimization (GEPA/MIPRO) | Teams already running MLflow for classical ML experiment tracking | https://mlflow.org/ |

---

## 8. Guardrails, Security & Red-Teaming

No single tool covers every risk — enterprises should assume **defense-in-depth** across multiple layered controls.

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **NVIDIA NeMo Guardrails** | Programmable conversational "rails" via a dedicated modeling language; topic control, jailbreak/content-safety microservices | Agents needing explicit dialogue-policy boundaries | https://github.com/NVIDIA/NeMo-Guardrails |
| **Guardrails AI** | Open-source validator framework for structured input/output enforcement against schemas/policies | Agents that must return structured, schema-valid outputs (forms, API payloads) | https://www.guardrailsai.com/ |
| **Lakera Guard** | Managed, adversarially-trained detection for prompt injection, jailbreaks, PII leakage, off-task tool calls | Enterprise SLA requirements for prompt-injection defense | https://www.lakera.ai/ |
| **LLM Guard** | Comprehensive open-source scanning: toxicity, PII, prompt injection | Self-hosted, privacy-conscious broad-coverage scanning | https://github.com/protectai/llm-guard |
| **Garak** | Open-source LLM vulnerability scanner ("nmap for LLMs") | Automated red-teaming of models and agent pipelines pre-release | https://github.com/leondz/garak |
| **Microsoft PyRIT** | Python Risk Identification Toolkit for automated red-teaming | CI-integrated, repeatable adversarial testing tied to Azure workflows | https://github.com/Azure/PyRIT |
| **AWS Bedrock Guardrails / Azure AI Content Safety** | Cloud-native, managed content-filtering guardrails integrated into the hyperscaler platform | Teams wanting guardrails as a managed platform feature rather than a separate tool | https://aws.amazon.com/bedrock/guardrails/ · https://azure.microsoft.com/en-us/products/ai-services/ai-content-safety |

**Key reference:** the **OWASP Top 10 for LLM Applications** and the emerging **OWASP Top 10 for Agentic Applications** are the standard threat maps guardrail coverage should be measured against. https://genai.owasp.org/

---

## 9. Workflow Automation & Durable Execution

Agents fail in production for reasons demos never surface: retries, partial failures, long-running human-approval waits. This layer makes agent execution **durable and recoverable**.

| Tool | Specialization | Best fit | Reference |
|---|---|---|---|
| **Temporal** | Durable execution engine: automatic retries, state persistence across failures/restarts, human-in-the-loop waits | Production agent workflows where "resume exactly where it failed" is a hard requirement (financial transactions, claims processing) | https://temporal.io/ |
| **n8n** | Visual, low-code workflow automation with 400+ integrations, AI-agent nodes | Business-process automation combining agents with traditional system integrations; citizen-developer-friendly | https://n8n.io/ |
| **Apache Airflow** | Batch/DAG-based workflow orchestration | Scheduled, data-pipeline-adjacent agent workflows (e.g., nightly agentic report generation) | https://airflow.apache.org/ |

---

## 10. Governance, Risk & Compliance

The layer every regulated enterprise must not skip. **No framework listed here was designed specifically for agentic AI** — all require extension to cover multi-agent cascading failures, scope creep, and action attribution.

| Framework / Standard | Type | What it governs | Reference |
|---|---|---|---|
| **NIST AI Risk Management Framework (AI RMF 1.0)** | Voluntary (US) | Four functions — Govern, Map, Measure, Manage; the de facto US enterprise risk-management baseline | https://www.nist.gov/itl/ai-risk-management-framework |
| **ISO/IEC 42001:2023** | Certifiable management-system standard | AI Management System (AIMS), analogous to ISO 9001/27001; third-party certification path | https://www.iso.org/standard/42001.html |
| **EU AI Act (Regulation (EU) 2024/1689)** | Binding regulation | Risk-based obligations; high-risk system requirements (conformity assessment, technical documentation, post-market monitoring) | https://eur-lex.europa.eu/eli/reg/2024/1689/oj/eng |
| **OWASP GenAI Security Project (Top 10 for LLM & Agentic Apps)** | Technical threat framework | Prompt injection, insecure output handling, excessive agency, and agent-specific risk categories | https://genai.owasp.org/ |
| **OECD AI Principles** | Voluntary, international | High-level values statement often used as the top-level governance anchor | https://oecd.ai/en/ai-principles |

**Practical guidance:** most mature enterprise programs run **NIST AI RMF** as the operating model, **ISO 42001** as the certifiable proof point for customers/partners, and map both against the **EU AI Act** if operating in or serving the EU. Build the evidence trail (logs, approval records, model/prompt version history) once — it satisfies all three.

---

## 11. Domain-Specific Considerations

### Telecom
- **TM Forum Open APIs / Open Digital Architecture (ODA)** — standard interfaces for OSS/BSS integration; increasingly used as the tool-calling contract layer for network and billing agents. https://www.tmforum.org/
- **GSMA** guidance on responsible AI and network automation. https://www.gsma.com/
- Architectural implication: agents that touch network configuration or customer billing need **hard-scoped, least-privilege tool access** (via MCP servers wrapping OSS/BSS APIs) and mandatory human approval for any state-changing network action.

### Healthcare
- **HIPAA** — governs PHI handling; any memory store, vector DB, or trace/observability tool touching patient data must have a signed Business Associate Agreement (BAA) and encryption at rest/in transit. https://www.hhs.gov/hipaa/index.html
- **HL7 FHIR** — standard for clinical data exchange; the natural schema for MCP tool servers connecting agents to EHR systems. https://www.hl7.org/fhir/
- **HITRUST CSF** — common healthcare-sector security certification layered on top of HIPAA. https://hitrustalliance.net/
- Architectural implication: prefer vector DBs / memory layers with clear BAA-eligible managed offerings (or self-hosted, VPC-isolated deployments); avoid sending PHI through gateways/observability tools without contractual and technical data-handling guarantees.

### Finance
- **PCI DSS** — governs payment-card data handling; relevant to any agent touching transaction data. https://www.pcisecuritystandards.org/
- **Federal Reserve/OCC/FDIC SR 26-2** (supersedes SR 11-7/SR 21-8) — US interagency model risk management guidance, the primary supervisory standard agent-based decisioning must satisfy in banking. https://www.federalreserve.gov/supervisionreg/srletters/
- **SOC 2** — common trust-services attestation for the vendors (gateways, observability, memory) in the stack. https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services
- Architectural implication: durable execution (Temporal) and immutable audit logging are not optional for agents that initiate or approve financial transactions — "who owns this when it fails in production" must have a documented, tested answer before go-live.

---

## 12. Tool Selection by Persona

| Persona | Primary concerns | Tools they touch most |
|---|---|---|
| **Solution Architect** | Stack design, build-vs-buy, vendor lock-in, TCO | Hyperscaler platform choice, orchestration framework, LLM gateway, protocol strategy (MCP/A2A) |
| **AI/Agent Engineer** | Building and iterating on agent logic | LangGraph/CrewAI/Semantic Kernel, memory frameworks, vector DB, evaluation frameworks |
| **Platform / DevOps Engineer** | Deployment, scaling, cost, reliability | LLM gateway, Temporal/n8n, observability platform, IaC for the hosting cloud |
| **Data Engineer** | Knowledge grounding, RAG pipelines, metric consistency | Vector DB, embedding pipelines, data connectors (MCP servers), governed semantic layer (Cube, dbt Semantic Layer) |
| **Analytics / BI Lead** | One source of truth for metrics across dashboards and agents | Semantic layer tooling, lineage/catalog (Atlan), row-level access policy |
| **QA / Test Engineer** | Regression prevention, CI gating | DeepEval, Promptfoo, Ragas, trajectory-level agent evals |
| **Security Engineer** | Attack surface, adversarial robustness | Lakera Guard, NeMo Guardrails, Garak, PyRIT, LLM Guard |
| **Governance / Compliance Lead** | Auditability, regulatory mapping, sign-off | NIST AI RMF, ISO 42001 mapping, audit-log/observability exports, human-approval workflow design |
| **Product Owner / IT Leader** | Business value, risk exposure, rollout pace | Enterprise agent platform dashboards, cost reporting from the gateway, eval scorecards |

---

## 13. A Practical Starting Stack

For a team starting an enterprise agentic build today, without prior platform commitment:

1. **Cloud/platform**: match the org's existing hyperscaler (Azure AI Foundry / Bedrock AgentCore / Vertex AI Agent Builder / watsonx)
2. **Orchestration**: LangGraph (max control) or CrewAI (fastest prototype) — or the native Microsoft Agent Framework / Strands SDK if already hyperscaler-committed
3. **Gateway**: LiteLLM (self-hosted, open) or Portkey (managed, governed)
4. **Memory/Knowledge**: pgvector or Qdrant for retrieval; Zep or Mem0 for session/long-term memory
5. **Semantic layer**: Cube or the dbt Semantic Layer in front of any warehouse an agent can query — before the agent goes anywhere near production data, not after
6. **Integration**: MCP for all tool/data connectors; A2A only if crossing an organizational boundary
7. **Observability**: Langfuse (self-hosted/open) or LangSmith (fastest setup)
8. **Evaluation**: DeepEval in CI, Promptfoo for red-teaming
9. **Guardrails**: platform-native guardrails (Bedrock Guardrails / Azure AI Content Safety) + Lakera or NeMo Guardrails for defense-in-depth
10. **Durable execution**: Temporal for any workflow with financial, clinical, or network state changes
11. **Governance**: map every above decision to NIST AI RMF from day one; layer ISO 42001 and EU AI Act obligations as regulatory exposure requires

---

## Sources & Further Reading

- LangChain, *AI Agent Frameworks* — https://www.langchain.com/resources/ai-agent-frameworks
- Microsoft, *Agent Framework documentation* — https://learn.microsoft.com/agent-framework
- OpenObserve, *LLM Observability Tools 2026* — https://openobserve.ai/blog/llm-observability-tools/
- MarkTechPost, *Top LLM Observability and Evaluation Platforms 2026* — https://www.marktechpost.com/2026/08/09/top-llm-observability-and-evaluation-platforms-in-2026-langfuse-langsmith-braintrust-arize-and-more-compared/
- Firecrawl, *Best Vector Databases 2026* — https://www.firecrawl.dev/blog/best-vector-databases
- EM360Tech, *Top 10 Security Tools for Agentic Systems* — https://em360tech.com/top-10/security-tools-for-agentic-systems
- AppSecSanta, *Best AI Security Tools 2026* — https://appsecsanta.com/ai-security-tools
- Redis, *MCP vs A2A: Which Protocol Do You Need?* — https://redis.io/blog/mcp-vs-a2a-which-protocol-do-you-need/
- Fastio, *8 Best Enterprise AI Platforms 2026* — https://fast.io/resources/best-enterprise-ai-platforms-2026/
- Atlan, *Best AI Agent Memory Frameworks 2026* — https://atlan.com/know/best-ai-agent-memory-frameworks-2026/
- NeuralTrust, *AI Governance Frameworks Compared* — https://neuraltrust.ai/blog/ai-governance-framework-comparison
- TrueFoundry, *A Definitive Guide to AI Gateways 2026* — https://www.truefoundry.com/blog/a-definitive-guide-to-ai-gateways-in-2026-competitive-landscape-comparison
- DeepEval, *Top 5 LLM Evaluation Frameworks 2026* — https://deepeval.com/blog/top-5-llm-evaluation-frameworks
- OWASP GenAI Security Project — https://genai.owasp.org/
- NIST AI Risk Management Framework — https://www.nist.gov/itl/ai-risk-management-framework
- ISO/IEC 42001:2023 — https://www.iso.org/standard/42001.html
- EU AI Act (Regulation (EU) 2024/1689) — https://eur-lex.europa.eu/eli/reg/2024/1689/oj/eng
- Cube, *Semantic Layer for AI Agents* — https://cube.dev/articles/semantic-layer-for-ai-agents-2026
- Atlan, *Best Semantic Layer Tools for BI and AI Agents* — https://atlan.com/know/best-semantic-layer-tools/
