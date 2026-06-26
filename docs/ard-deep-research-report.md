# 🧠 ARD (Agentic Resource Discovery) Protocol — In-Depth Research Report

> **Version**: v1.0 | **Date**: 2026-06-26 | **Category**: Technical Research / Industry Analysis

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Background & Origins](#2-background--origins)
3. [Protocol Architecture & Technical Details](#3-protocol-architecture--technical-details)
4. [Ecosystem & Current Adoption Status](#4-ecosystem--current-adoption-status)
5. [Competitive Landscape Analysis](#5-competitive-landscape-analysis)
6. [ARD's Relationship with MCP and A2A](#6-ards-relationship-with-mcp-and-a2a)
7. [Trust & Security Model](#7-trust--security-model)
8. [Challenges & Limitations](#8-challenges--limitations)
9. [Future Outlook & Roadmap](#9-future-outlook--roadmap)
10. [Actionable Recommendations](#10-actionable-recommendations)
11. [References](#11-references)

---

## 1. Executive Summary

**Agentic Resource Discovery (ARD)** is an open protocol specification jointly launched by **11 major technology companies** — **Google, Microsoft, Hugging Face, GitHub, NVIDIA, Snowflake, Salesforce, Cisco, Databricks, GoDaddy, and ServiceNow**. The draft v0.9 was officially published on **June 17, 2026**, hosted under the **AI Catalog Working Group** of the **Linux Foundation** under the **Apache 2.0** license.

ARD's core mission is to provide a standard **runtime resource discovery mechanism** for AI agents — enabling agents to dynamically discover tools, MCP servers, A2A agents, APIs, and other capabilities through natural language queries, just as humans use search engines, instead of relying on pre-installed or hard-coded configurations.

This report provides an in-depth analysis of the ARD protocol from multiple dimensions: protocol background, technical architecture, ecosystem status, competitive comparison, trust model, challenges, and future outlook, along with actionable recommendations.

---

## 2. Background & Origins

### 2.1 The "Babel" Dilemma of the Agent Ecosystem

In 2025–2026, the AI agent space experienced explosive growth but faced a serious **interoperability crisis**:

- **MCP (Model Context Protocol)**: Anthropic's tool-connection protocol, standardizing how agents invoke tools
- **A2A (Agent-to-Agent)**: Google's inter-agent communication protocol, standardizing how agents collaborate
- **Missing the critical piece**: How do agents **discover** available tools and other agents?

The reality was that developers had to manually configure each agent's tool list. When the tool count grows from a handful to hundreds or thousands, this model becomes completely unsustainable.

### 2.2 The Birth of ARD

On June 17, 2026, Google formally announced the ARD specification on its official developer blog, immediately gaining endorsements from 10 industry giants. Multiple media outlets described this as "the birth of a search engine for AI agents."

**Key Timeline:**

| Date | Event |
|------|-------|
| November 2024 | Anthropic releases MCP protocol |
| April 2025 | Google releases A2A protocol (50+ partners) |
| July 2025 | Cisco donates AGNTCY project to Linux Foundation |
| June 17, 2026 | ARD v0.9 draft officially released |
| June 2026 | Hugging Face releases hf-discover tool |
| June 2026 | DataRobot publishes ARD-compatible Agent Skills catalog |
| 2026 (coming months) | Google Agent Registry plans native ARD integration |

### 2.3 Strategic Significance — Industry Landscape

ARD's launch is widely seen as **an ecosystem containment strategy by traditional enterprise software giants against OpenAI and Anthropic**.

- **Neither OpenAI nor Anthropic** appear on the initial supporter list
- ARD's 11 backers span cloud computing (Google Cloud, Microsoft Azure), developer platforms (GitHub), AI community (Hugging Face), enterprise software (Salesforce, ServiceNow), data infrastructure (Snowflake, Databricks), hardware (NVIDIA), networking (Cisco), and internet services (GoDaddy)
- These companies have formed an **open but federated** ecosystem alliance aimed at preventing the agent ecosystem from being locked in by any single company

---

## 3. Protocol Architecture & Technical Details

### 3.1 Core Design Principles

| Principle | Description |
|-----------|-------------|
| **Search-First Discovery** | Agents discover capabilities dynamically through search, not pre-installation |
| **Scalability Beyond Context Windows** | Moves selection logic outside the LLM context window into a dedicated search service |
| **Artifact Agnostic Envelope** | Uses IANA Media Types to identify resource types without constraining internal schemas |
| **Value-or-Reference** | Entries must contain either `url` (remote reference) or `data` (inline) — never both or neither |
| **REST API Baseline** | Registries must expose an HTTP REST search interface |

### 3.2 Two Core Primitives

ARD defines two complementary primitives:

#### ① Catalog

An organization publishes a static manifest file at `/.well-known/ai-catalog.json` on its domain, listing all the AI capabilities it provides.

**Catalog file structure** (based on JSON Schema):

```json
{
  "specVersion": "1.0",
  "host": {
    "displayName": "Example Corp",
    "identifier": "did:web:example.com",
    "documentationUrl": "https://docs.example.com/ard",
    "trustManifest": { ... }
  },
  "entries": [
    {
      "identifier": "urn:air:example.com:server:search-mcp",
      "displayName": "Search MCP Server",
      "type": "application/mcp-server-card+json",
      "url": "https://example.com/.well-known/mcp-server-card.json",
      "description": "Full-text search service supporting semantic retrieval and keyword matching",
      "tags": ["search", "semantic", "fulltext"],
      "capabilities": ["SearchTool", "IndexTool"],
      "representativeQueries": [
        "search for documents about AI agents",
        "find the latest research papers"
      ],
      "version": "1.2.0",
      "updatedAt": "2026-06-17T00:00:00Z"
    }
  ]
}
```

**Key Field Reference:**

| Field | Required | Description |
|-------|----------|-------------|
| `specVersion` | ✅ | Must be `"1.0"` (note: protocol is v0.9 draft but Schema version is 1.0) |
| `host.displayName` | ✅ | Human-readable publisher name |
| `host.identifier` | No | Verifiable identifier (e.g., `did:web:domain.com`) |
| `entries[].identifier` | ✅ | URN-format unique identifier (`urn:air:<publisher>:<namespace>:<name>`) |
| `entries[].type` | ✅ | IANA Media Type identifying resource type |
| `entries[].url` or `data` | ✅ (one of) | Remote reference or inline data |
| `entries[].representativeQueries` | No | 2-5 representative search queries (for vector indexing and retrieval) |

#### ② Registry

A Registry is the "search engine for agents" — it crawls Catalogs across domains, builds an index, and provides runtime search via a `POST /search` API.

**Registry search request example:**

```http
POST /search HTTP/1.1
Content-Type: application/json

{
  "query": "find an MCP server for database query",
  "type": "application/mcp-server-card+json",
  "limit": 10
}
```

**Registry search response example:**

```json
{
  "results": [
    {
      "identifier": "urn:air:example.com:server:db-mcp",
      "displayName": "Database Query MCP Server",
      "type": "application/mcp-server-card+json",
      "url": "https://example.com/cards/db-mcp.json",
      "score": 0.92,
      "description": "Natural language database query interface",
      "tags": ["database", "sql", "query"]
    }
  ],
  "totalResults": 1
}
```

### 3.3 Supported Resource Types (IANA Media Types)

| Media Type | Meaning | Example Scenarios |
|------------|---------|-------------------|
| `application/mcp-server-card+json` | MCP Server | Database query tool, file processing tool |
| `application/a2a-agent-card+json` | A2A Agent | Customer service agent, data analysis agent |
| `application/ai-skill` | AI Skill | Model training skill, data preprocessing skill |
| `application/ai-catalog+json` | Nested sub-catalog | Hierarchical catalogs for large organizations |
| `application/ai-registry+json` | Searchable dynamic Registry | Hugging Face Discover, GitHub Agent Finder |
| `application/vnd.oai.openapi+json` | OpenAPI description | REST API interfaces |

### 3.4 URN Naming Convention

ARD uses RFC 8141-compliant URN format:

```
urn:air:<publisher>:<namespace>:<agent-name>
```

Where:
- `air` = Fixed namespace identifier for Agentic Resource
- `publisher` = Publisher domain or DID (e.g., `huggingface.co`, `github.com:datarobot-oss`)
- `namespace` = Resource category (e.g., `registry`, `server`, `agent`, `skill`, `api`)
- `agent-name` = Specific resource name (e.g., `discover`, `blog-mcp`, `model-training`)

**Real-world examples:**

| URN | Owner |
|-----|-------|
| `urn:ai:huggingface.co:registry:discover` | Hugging Face Discover Registry |
| `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:model-training` | DataRobot Model Training Skill |
| `urn:ai:suganthan.com:server:blog-mcp` | Suganthan Blog MCP Server |
| `urn:ai:joost.blog:a2a:ask-joost` | Joost de Valk's A2A Agent |

### 3.5 Discovery Flow (End-to-End)

```
Agent ──(1) Natural Language Query──▶ Registry
                                       │
                                 (2) Search Index
                                       │
                                       ▼
                                 ┌──────────────┐
                                 │ Index Store   │
                                 │(Vector+Keyword)│
                                 └──────────────┘
                                       │
                                 (3) Ranked Results
                                       │
                                       ▼
Agent ◀──(4) Ranked Results w/ Trust Metadata─── Registry
  │
  │ (5) Select target from results
  │ (6) Fetch detailed card via target URL
  │ (7) Verify publisher identity (did:web / JWS)
  ▼
Target MCP Server / A2A Agent / API
  │
  │ (8) Invoke via native protocol (MCP/A2A/REST)
  ▼
Capability Execution
```

---

## 4. Ecosystem & Current Adoption Status

### 4.1 Organizations with Published Public Catalogs

As of 2026-06-26, **4 organizations/individuals** have published public `ai-catalog.json` files:

| # | Publisher | Type | Entries | Resource Types | Advanced Features |
|---|-----------|------|---------|----------------|-------------------|
| 1 | **Hugging Face** (huggingface.co) | Enterprise | 1 (Registry) | `ai-registry` | Points to dynamic Discover Registry |
| 2 | **DataRobot** (datarobot.com) | Enterprise | 10 | `ai-skill` | Full MLOps lifecycle coverage |
| 3 | **Suganthan Mohanadasan** (suganthan.com) | Individual | 3 | `a2a-agent`, `mcp-server`, `openapi` | `representativeQueries`, `capabilities`, `version`, `did:web` |
| 4 | **Joost de Valk** (joost.blog) | Individual | 2 | `mcp-server`, `a2a-agent` | `did:web`, JWS-signed `trustManifest` |

### 4.2 Organizations That Have Announced Support But Not Yet Published

| Organization | Planned Integration | Expected Timeline |
|-------------|-------------------|-------------------|
| **Google** | Agent Registry (part of Gemini Enterprise Agent Platform) | Coming months |
| **GitHub** | Copilot Agent Finder | Partially live |
| **Cisco** | AGNTCY Agent Directory (as ARD reference implementation) | Open-sourced |
| **Salesforce** | Native ARD integration into Salesforce Platform | TBD |
| **Snowflake** | Discovery via CoWork and CoCo | TBD |

### 4.3 Reference Implementations & Tools

| Tool | Developer | Description | URL |
|------|-----------|-------------|-----|
| **hf-discover** | Hugging Face | ARD client CLI + server; supports search, navigate, self-hosted Registry | [GitHub](https://github.com/huggingface/hf-discover) |
| **AGNTCY Directory** | Cisco / Linux Foundation | OCI + DHT-based distributed Agent Directory service | [GitHub](https://github.com/agntcy) |
| **ard-spec** | ARD Working Group | Official spec repository with JSON Schema, CDDL, URN guide | [GitHub](https://github.com/ards-project/ard-spec) |
| **ARD Lab** | Agent Field | End-to-end hands-on tutorial (cross-org data analytics example) | [GitHub](https://github.com/Agent-Field/agentic-resource-discovery-lab) |
| **ARD Quickstart** | Google | Official quickstart site | [agenticresourcediscovery.org](https://agenticresourcediscovery.org) |

### 4.4 Accessibility Scan Results

Scan of `/.well-known/ai-catalog.json` across 11 founding organizations:

| Organization | Domain | HTTP Status | Notes |
|-------------|--------|-------------|-------|
| ✅ Hugging Face | huggingface.co | 200 | Published |
| ✅ DataRobot | datarobot.com | 200 | Published |
| ✅ Suganthan | suganthan.com | 200 | Published |
| ✅ Joost de Valk | joost.blog | 200 | Published |
| ⬜ Google | google.com | 404 | Not published |
| ⬜ Google Cloud | cloud.google.com | 404 | Not published |
| ⬜ Microsoft | microsoft.com | 404 | Not published |
| ⬜ GitHub | github.com | 404 | Not published |
| ⬜ Snowflake | snowflake.com | 404 | Not published |
| ⬜ Salesforce | salesforce.com | No response | Not published |
| ⬜ Databricks | databricks.com | 404 | Not published |
| ⬜ NVIDIA | nvidia.com | 403 | Not publicly accessible |
| ⬜ GoDaddy | godaddy.com | 403 | Not publicly accessible |
| ⬜ ServiceNow | servicenow.com | 403 | Not publicly accessible |
| ⬜ Cisco | cisco.com | TBD | Not scanned |

> **Conclusion**: The ARD ecosystem is in its **very early stages**. Most founding organizations have not yet published public Catalogs. This presents both a challenge and an opportunity — early movers will gain ecosystem positioning advantages.

---

## 5. Competitive Landscape Analysis

### 5.1 Agent Discovery/Registry Solutions — Full Comparison

| Solution | Type | Governance | Trust Model | Use Case |
|----------|------|------------|-------------|----------|
| **ARD (Catalogs + Registries)** | Open Standard | Linux Foundation | Domain-anchored + optional did:web | General cross-org discovery |
| **MCP Registry** | Centralized | GitHub | GitHub OAuth + DNS TXT | MCP server discovery |
| **A2A Agent Cards** | Decentralized | No central authority | TLS + OAuth2 | Agent self-description |
| **AGNTCY Directory** | Distributed P2P | Linux Foundation | OCI signing + DHT | Enterprise federated directory |
| **NANDA AgentFacts** | Decentralized Verifiable | MIT AIDE | W3C VC verifiable credentials | Privacy-preserving credential-based discovery |
| **Microsoft Entra Agent ID** | Enterprise SaaS | Microsoft | Azure AD / Zero Trust | Microsoft ecosystem enterprise management |
| **agent:// Protocol** | URI Scheme | IETF (Draft) | Depends on underlying implementation | General agent addressing |

### 5.2 ARD's Unique Positioning

ARD's differentiating advantages:

1. **Coalition backing**: Jointly launched by 11 industry giants, not controlled by a single vendor
2. **Domain as trust**: Leverages mature DNS infrastructure as trust anchor, no additional PKI needed
3. **Protocol-agnostic**: Does not replace MCP/A2A, but works alongside them as a discovery layer
4. **Value-or-Reference model**: Flexible support for both inline data and remote references
5. **Unified search semantics**: Semantic search via `representativeQueries`

### 5.3 Key Weaknesses

1. **Very early ecosystem**: Most founding organizations have not yet deployed
2. **Lack of production-grade Registry**: No large-scale public Registry in operation yet
3. **No mandatory trust mechanism**: `trustManifest` is optional, early Catalogs generally omit it
4. **Schema version confusion**: Protocol version is v0.9 but Schema field requires `"1.0"`
5. **OpenAI and Anthropic absence**: Two key players are not on board, which may lead to standard fragmentation

---

## 6. ARD's Relationship with MCP and A2A

### 6.1 Three-Layer Protocol Stack Model

The industry is converging on a consensus — ARD, MCP, and A2A are not competitors but **three complementary layers** of agent interoperability:

```
  ┌─────────────────────────────────┐
  │   A2A (Agent-to-Agent)          │  ← Horizontal: how agents communicate
  │   Google-led                    │
  ├─────────────────────────────────┤
  │   ARD (Agentic Resource Disc.)  │  ← Discovery: where to find capabilities
  │   Linux Foundation hosted       │
  ├─────────────────────────────────┤
  │   MCP (Model Context Protocol)  │  ← Tool access: how agents invoke tools
  │   Anthropic-led                 │
  └─────────────────────────────────┘
```

### 6.2 Network Protocol Analogy

| AI Agent Layer | Network Protocol Analogy | Description |
|----------------|-------------------------|-------------|
| **MCP** | Layer 2 (Data Link) | Provides detailed tool-level connectivity, limited scalability |
| **ARD** | DNS + Search Engine | Decouples "where is the resource" from context window into search service |
| **A2A** | Layer 3 (Routing) | Aggregates capability boundaries, provides cross-domain collaboration gateway |

### 6.3 End-to-End Collaboration Flow

Using the example of "an enterprise smart assistant automating the expense reimbursement process":

```
1. Agent discovers the existence of a "Finance Assistant" via A2A Agent Card
2. Agent searches via ARD to find an "Expense Report MCP Server"
3. Agent connects to and invokes the MCP Server via the MCP protocol
4. MCP Server returns query results
5. Agent passes the results to an "Approval Agent" via A2A for next-step processing
```

### 6.4 Selection Guide

| Scenario | Recommended Approach |
|----------|---------------------|
| Single agent needing external tools | MCP alone suffices |
| Multi-agent system requiring orchestration | A2A + MCP |
| Cross-org discovery of agents and tools | **ARD + MCP/A2A** |
| Enterprise agent directory + governance | ARD (AGNTCY) + Microsoft Entra |
| Individual developer publishing personal agents | ARD Catalog (simplest approach) |

---

## 7. Trust & Security Model

### 7.1 Layered Trust Architecture

ARD's trust model is structured in three layers:

#### Layer 1: Domain Anchoring (Basic)

- Publishers prove basic identity through domain ownership
- Catalogs served over HTTPS, relying on TLS certificate validation
- Fixed resolution path at `/.well-known/ai-catalog.json`

#### Layer 2: `did:web` Identity (Enhanced)

- Uses `did:web:domain.com` as `host.identifier`
- No blockchain required, directly leverages DNS infrastructure
- DID document published at `/.well-known/did.json`
- Adopted by Suganthan and Joost de Valk

#### Layer 3: `trustManifest` (Highest Level)

An optional cryptographic security envelope providing:

| Component | Description |
|-----------|-------------|
| `identity` | Globally unique cryptographic identity (SPIFFE ID or DID URI) |
| `identityType` | Format hint (spiffe / did / https / other) |
| `trustSchema` | Trust framework identifier and version |
| `attestations` | Verifiable claims (SOC 2, HIPAA compliance audits) |
| `provenance` | Cryptographic lineage tracking (derived from, published from) |
| `signature` | Detached JWS signature over trustManifest fields |

### 7.2 Security Considerations

| Threat | ARD Mitigation |
|--------|----------------|
| Forged Catalog | Domain verification + optional JWS signing |
| Man-in-the-Middle | HTTPS transport encryption |
| Malicious Registry | Anyone can run a Registry, but Catalog sources are independently verifiable |
| Privacy leakage | Catalogs contain only discovery metadata, not actual data |
| Stale resources | `updatedAt` field + periodic Registry re-crawling |

### 7.3 Current Implementation Status

- **Early Catalogs (Hugging Face, DataRobot)**: Basic domain anchoring only
- **Intermediate Catalog (Suganthan)**: Uses `did:web` identity
- **Advanced Catalog (Joost de Valk)**: Uses `did:web` + JWS-signed `trustManifest`

---

## 8. Challenges & Limitations

### 8.1 Technical Challenges

| Challenge | Detail | Severity |
|-----------|--------|----------|
| **Cold start problem** | Insufficient Catalogs → no search value → no one publishes → vicious cycle | 🔴 High |
| **Search quality** | Pure semantic search accuracy in specialized domains (medical, legal) not yet validated | 🟡 Medium |
| **Real-time freshness** | Latency between Catalog update and Registry re-indexing | 🟡 Medium |
| **Version compatibility** | Protocol v0.9 but Schema version requires "1.0" — a mismatch | 🟢 Low |
| **Cross-Registry queries** | Federated query standards and performance across distributed Registries undefined | 🟡 Medium |
| **Malicious Catalog defense** | No built-in mechanism to prevent publishing malicious/fake Catalog entries | 🔴 High |

### 8.2 Ecosystem Challenges

| Challenge | Detail |
|-----------|--------|
| **OpenAI and Anthropic absence** | If both key players launch competing standards, ecosystem fragmentation may follow |
| **Standard maturity** | v0.9 draft — breaking changes possible in future versions |
| **Enterprise adoption willingness** | Trust in "open standards" takes time to build in enterprise settings |
| **Individual developer barrier** | Requires deploying files on a domain root, potentially excluding individual developers |
| **Legacy system integration** | Enterprises need to retrofit existing infrastructure to publish Catalogs |

### 8.3 Governance Challenges

- Linux Foundation working group governance model has relatively slow decision-making
- 11 founding companies have diverse interests — long-term direction may diverge
- No clear certification/compliance mechanism to ensure Registry behavior standards

---

## 9. Future Outlook & Roadmap

### 9.1 Short-Term (2026 H2)

| Milestone | Expected | Description |
|-----------|----------|-------------|
| v1.0 stable release | 2026 Q3-Q4 | Iterated based on v0.9 feedback |
| Google Agent Registry ARD support | Coming months | Native ARD integration in Gemini Enterprise Agent Platform |
| GitHub Copilot Agent Finder maturity | In progress | Expanding discovery for more MCP servers |
| More founding orgs publish Catalogs | 2026 H2 | Google, Microsoft, Salesforce expected to publish |
| hf-discover ongoing iteration | In progress | Adding more Registry compatibility |

### 9.2 Medium-Term (2027)

| Direction | Description |
|-----------|-------------|
| **Large-scale public Registry** | ARD Registries comparable to web search engines emerge |
| **Cross-Registry federated queries** | Standardized distributed Registry query protocol |
| **Enterprise trust framework** | Deep SPIFFE/SPIRE integration, zero-trust architecture support |
| **Developer toolchain maturity** | CLI, SDK, CI/CD integration tooling ecosystem matures |
| **Open-source Registry implementations** | More self-hosted open-source Registries beyond AGNTCY |

### 9.3 Long-Term (2028+)

| Vision | Description |
|--------|-------------|
| "Google for Agents" | Registries become agent infrastructure, like search engines for the web |
| Everything discoverable | MCP servers, A2A agents, AI Skills, APIs, datasets, and more |
| Industry-vertical Registries | Specialized Registries for healthcare, finance, legal, etc. |
| Standard unification | ARD becomes the de facto standard for agent discovery |

---

## 10. Actionable Recommendations

### 10.1 For This Repository (ard-catalogs)

| Priority | Action | Description |
|----------|--------|-------------|
| 🔴 High | Continuously track founding orgs' Catalog release status | Set up automated monitoring; ingest new Catalogs promptly |
| 🔴 High | Watch for Google, GitHub, Salesforce Catalog launches | These three will drive ecosystem tipping points |
| 🟡 Medium | Add Schema validation for Catalogs | Use ajv-cli with official Schema to auto-validate collected files |
| 🟡 Medium | Add heartbeat-based Catalog health checks | Use hf-discover or container-based periodic accessibility checks |
| 🟢 Low | Incorporate representative queries into INDEX.md | Enhance index searchability and usability |

### 10.2 For Teams Planning to Adopt ARD

| Phase | Action |
|-------|--------|
| **Phase 1: Learn** | Read the ARD spec; understand Catalog and Registry concepts |
| **Phase 2: Publish a Catalog** | Deploy `ai-catalog.json` on your own domain; consider using did:web |
| **Phase 3: Integrate** | Incorporate ARD search into your agents (via hf-discover or other Registry) |
| **Phase 4: Contribute** | Submit a PR to this repository to include your Catalog in the global index |

### 10.3 Risk Warnings

1. **Standard fragmentation risk**: If OpenAI/Anthropic launch competing standards, evaluate compatibility strategies
2. **Upgrade risk**: v0.9 → v1.0 may include breaking changes; track spec changes closely
3. **Security risk**: Without `trustManifest`, be cautious about resources returned from Registries
4. **Dependency risk**: Currently, Hugging Face's Discover is the only available Registry — a single point of dependency

---

## 11. References

### Official Specifications & Documentation

- [ARD Official Specification](https://github.com/ards-project/ard-spec) — Complete ARD protocol specification
- [ARD JSON Schema](https://github.com/ards-project/ard-spec/blob/main/spec/schemas/ai-catalog.schema.json) — JSON Schema definition for ai-catalog.json
- [ARD Quickstart](https://agenticresourcediscovery.org) — Official quickstart site
- [ai-catalog Base Specification](https://github.com/Agent-Card/ai-catalog) — Underlying catalog format specification

### Official Announcements

- [Google Developers Blog: Announcing the ARD Specification](https://developers.googleblog.com/en/announcing-the-agentic-resource-discovery-specification/) (2026-06-17)
- [Hugging Face Blog: Let agents search](https://huggingface.co/blog/agentic-resource-discovery-launch) (2026-06-17)
- [Cisco Outshift Blog: How ARD helps agents find each other at enterprise scale](https://outshift.cisco.com/blog/ai-ml/agentic-resource-discover-specification-helps-agents-find-each-other)
- [Salesforce Blog: How Open Discovery Is Unifying the Agentic Enterprise](https://www.salesforce.com/blog/open-discovery-agentic-enterprise/)
- [Snowflake Blog: Snowflake and the ARD Specification](https://www.snowflake.com/en/blog/agentic-resource-discovery-specification/)

### Adoption Case Studies

- [DataRobot: Agent Skills Are Now Discoverable Through ARD](https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/)
- [Suganthan: Personal ARD Implementation Experience](https://suganthan.com/blog/agentic-resource-discovery/)
- [Joost de Valk: Integrating ARD with OKF](https://joost.blog/okf-ard/)

### Reference Implementations

- [Hugging Face hf-discover](https://github.com/huggingface/hf-discover) — ARD client/server
- [Cisco AGNTCY Agent Directory](https://github.com/agntcy) — ARD reference implementation
- [ARD Tutorial Lab](https://github.com/Agent-Field/agentic-resource-discovery-lab) — End-to-end ARD hands-on tutorial

### Media Coverage & Analysis

- [heise.de: Agentic Resource Discovery — Search engine for AI agents](https://www.heise.de/en/news/Agentic-Resource-Discovery-Search-engine-for-AI-agents-11338833.html)
- [Search Engine Journal: Google, Microsoft Back Draft AI Agent Discovery Spec](https://www.searchenginejournal.com/google-microsoft-back-draft-ai-agent-discovery-spec/579894/)
- [ZDNet: AI agents are getting their own search engine](https://www.zdnet.com/article/ai-agents-are-getting-their-own-search-engine/)
- [WinBuzzer: Google is Building a Trusted Directory for AI Agents](https://winbuzzer.com/2026/06/22/google-ard-release-sharpens-ai-agent-protocol-fight-xcxwbn/)
- [DigitalOcean: A2A vs MCP — How These AI Agent Protocols Actually Differ](https://www.digitalocean.com/community/tutorials/a2a-vs-mcp-ai-agent-protocols)
- [Stream Blog: Top AI Agent Protocols in 2026 — MCP, A2A, ACP & More](https://getstream.io/blog/ai-agent-protocols/)
- [Stackademic: ARD — The Missing Layer for AI Agents](https://blog.stackademic.com/agentic-resource-discovery-ard-the-missing-layer-for-ai-agents-29e2e7542fb7)

### Academic Papers

- [From Glue-Code to Protocols: AI Agent Interoperability (arXiv 2505.03864)](https://arxiv.org/pdf/2505.03864)
- [A Survey of AI Agent Registry Solutions (arXiv 2508.03095)](https://ar5iv.labs.arxiv.org/html/2508.03095)
- [The agent:// Protocol — IETF Draft](https://datatracker.ietf.org/doc/html/draft-narvaneni-agent-uri-03)

---

## Appendix

### A. Quick Checklist: Ready to Publish an ARD Catalog?

- [ ] Own a domain accessible via HTTPS
- [ ] Determined the resource types to publish (MCP Server / A2A Agent / AI Skill / API)
- [ ] Assigned a unique URN to each resource (following `urn:air:<publisher>:<namespace>:<name>`)
- [ ] Generated the `ai-catalog.json` file
- [ ] Validated the file against the official JSON Schema
- [ ] Deployed to `/.well-known/ai-catalog.json`
- [ ] (Optional) Set up `did:web` identity
- [ ] (Optional) Generated `trustManifest` signature
- [ ] Submitted a PR to include your Catalog in [ard-catalogs](https://github.com/huangxn29/ard-catalogs)

### B. Glossary

| Term | Description |
|------|-------------|
| ARD (Agentic Resource Discovery) | An open protocol for AI agent runtime capability discovery |
| Catalog | Static manifest published at `/.well-known/ai-catalog.json` |
| Registry | Indexed search service over Catalogs |
| MCP (Model Context Protocol) | Anthropic's protocol for agent-to-tool connectivity |
| A2A (Agent-to-Agent) | Google's protocol for inter-agent collaboration |
| URN (Uniform Resource Name) | Unique resource identifier in ARD (`urn:air:...`) |
| IANA Media Type | Resource type identifier (e.g., `application/mcp-server-card+json`) |
| trustManifest | Security metadata envelope with cryptographic signatures |
| did:web | Web-based DID method for verifiable domain identity |
| JWS (JSON Web Signature) | Digital signature format for trustManifest |
| SPIFFE/SPIRE | Secure identity framework for production workloads |
| AGNTCY | Cisco's open-source Agent Directory project, ARD reference implementation |

---

> **Document maintainer**: Research Agent (ARD Technical Researcher)
> **Last updated**: 2026-06-26
> **Next review**: Recommended when ARD v1.0 is released or when major founding organizations (Google, Microsoft) publish their Catalogs
