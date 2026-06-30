<!--
  ard-catalogs — ARD Catalog Bible
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  A curated collection of public ai-catalog.json manifests from across the web.
  English version.
-->

<p align="center">
  <img src="https://agenticresourcediscovery.org/favicon.ico" width="64" />
</p>

# 🗂️ ARD Catalog Bible

> Collect public high-quality `ai-catalog.json` files from across the web, making this repository the definitive "ARD Catalog Bible."

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![ARD Spec v0.9](https://img.shields.io/badge/ARD-v0.9%20(Draft)-purple)](https://github.com/ards-project/ard-spec)
[![Star History](https://img.shields.io/badge/Star-History-gold?logo=github)](https://star-history.com/#huangxn29/ard-catalogs&Date)
[![Follow @huangxn29](https://img.shields.io/badge/X-@huangxn29-000000?logo=x)](https://x.com/huangxn29)

[中文版 →](README.zh.md)

> **Follow on X ([@huangxn29](https://x.com/huangxn29))** for updates on new catalog additions, ARD ecosystem developments, and more.

---

## 📖 Introduction

**Agentic Resource Discovery (ARD)** is an open, federated discovery protocol backed by **Google, Microsoft, Hugging Face, GitHub, NVIDIA, Snowflake, Salesforce, Cisco, Databricks, GoDaddy, and ServiceNow**, hosted under the Linux Foundation. Draft v0.9 was announced on June 17, 2026.

ARD defines a standard way for AI agents to dynamically discover tools, MCP servers, A2A agents, and APIs via a machine-readable manifest hosted at `/.well-known/ai-catalog.json`.

This repository aims to:
- ✅ **Collect** publicly available `ai-catalog.json` files from the open web
- ✅ **Maintain** accurate copies with metadata
- ✅ **Index** provide a global lookup table for easy reference
- ✅ **Educate** include the official JSON Schema and example files
- ✅ **Track** monitor adoption progress across publishers

---

## 📂 Repository Structure

```
ard-catalogs/
├── catalogs/               # Collected ai-catalog.json files (organized by publisher)
│   ├── huggingface/        # Hugging Face — huggingface.co
│   ├── datarobot/          # DataRobot — datarobot.com
│   ├── suganthan/          # Suganthan (individual developer) — suganthan.com
│   └── joost.blog/         # Joost de Valk (Yoast founder) — joost.blog
├── docs/                   # Research reports and documentation
│   └── ard-deep-research-report.md  # In-depth ARD protocol research report
├── spec/                   # ARD protocol specification documents
│   └── schemas/            # JSON Schema definitions
├── examples/               # Example catalog files from the ARD spec
├── INDEX.md                # Global index
└── README.md               # This file
```

---

## 🌐 Collected Catalogs

| # | Publisher | Domain | Catalog URL | Resource Types | Collected |
|---|-----------|--------|-------------|----------------|-----------|
| 1 | **Hugging Face** | huggingface.co | [🔗](https://huggingface.co/.well-known/ai-catalog.json) | ai-registry | 2026-06-29 |
| 2 | **DataRobot** | datarobot.com | [🔗](https://datarobot.com/.well-known/ai-catalog.json) ⚠️ 403 | ai-skill | 2026-06-29 |
| 3 | **Suganthan Mohanadasan** | suganthan.com | [🔗](https://suganthan.com/.well-known/ai-catalog.json) | a2a-agent, mcp-server, openapi | 2026-06-29 |
| 4 | **Joost de Valk** | joost.blog | [🔗](https://joost.blog/.well-known/ai-catalog.json) | mcp-server, a2a-agent | 2026-06-29 |

> **Note**: ARD v0.9 draft was published on June 17, 2026. Currently only a few pioneering organizations and developers have published public ai-catalog.json files. This repository will be continuously updated as the ecosystem matures.

---

## 📋 Catalog Deep Dive

### 1. Hugging Face

- **URL**: `https://huggingface.co/.well-known/ai-catalog.json`
- **Type**: Registry
- **Description**: Hugging Face publishes an ARD entry pointing to its Discover Registry, which indexes Skills, Space skills, and MCP-enabled Spaces on Hugging Face
- **Identifier**: `urn:ai:huggingface.co:registry:discover`
- **Registry Endpoint**: `https://huggingface-hf-discover.hf.space`
- **CLI Tool**: `hf-discover` (open source, [GitHub](https://github.com/huggingface/hf-discover))

### 2. DataRobot

- **URL**: `https://datarobot.com/.well-known/ai-catalog.json`
- **Type**: Skills collection
- **Description**: DataRobot publishes 10 Agent Skills covering model training, deployment, prediction, feature engineering, monitoring, explainability, data preparation, CI/CD, external agent monitoring, and agent building
- All entries are `application/ai-skill` type
- **Reference**: [DataRobot Blog](https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/)

### 3. Suganthan Mohanadasan

- **URL**: `https://suganthan.com/.well-known/ai-catalog.json`
- **Type**: Mixed (A2A Agent + MCP Server + OpenAPI)
- **Description**: An individual developer's full ARD deployment with 3 entries:
  - A2A Blog Agent (blog search agent)
  - Blog MCP Server
  - Blog REST API (OpenAPI)
- Uses `representativeQueries`, `capabilities`, `version` and other advanced fields
- Uses `did:web` as the host identifier
- **Reference**: [Suganthan's blog](https://suganthan.com/blog/agentic-resource-discovery/)

### 4. Joost de Valk (joost.blog)

- **URL**: `https://joost.blog/.well-known/ai-catalog.json`
- **Type**: Mixed (MCP Server + A2A Agent)
- **Description**: Joost de Valk — founder of Yoast — publishes an ARD catalog with 2 entries:
  - MCP Server: search blog content about SEO, WordPress, AI, and the open web
  - A2A Agent: ask natural-language questions grounded in his published posts and talks
- Uses `did:web` identity with a `trustManifest` containing a JWS signature
- **Reference**: [Joost's blog](https://joost.blog/okf-ard/)

---

## 📐 Protocol Overview

| Property | Value |
|----------|-------|
| **Full Name** | Agentic Resource Discovery (ARD) |
| **Version** | v0.9 (Draft) |
| **Announced** | 2026-06-17 |
| **License** | Apache 2.0 |
| **Governance** | Linux Foundation (AI Catalog Working Group) |
| **Core File** | `/.well-known/ai-catalog.json` |
| **URN Format** | `urn:air:<publisher>:<namespace>:<agent-name>` |
| **Backers** | Google, Microsoft, Hugging Face, GitHub, NVIDIA, Snowflake, Salesforce, Cisco, Databricks, GoDaddy, ServiceNow |

### Core Design Principles

1. **Search-First Discovery**: Agents discover capabilities dynamically through search, not pre-installation
2. **Scalability Beyond Context Windows**: Moves selection outside the LLM context window into a dedicated search service
3. **Artifact Agnostic Envelope**: Uses IANA Media Types to identify resource types without constraining internal schemas
4. **Value-or-Reference**: Entries must contain either `url` (remote reference) or `data` (inline) — never both or neither
5. **REST API Baseline**: Registries must expose an HTTP REST search interface

### Resource Types (IANA Media Types)

| Type | Meaning |
|------|---------|
| `application/mcp-server-card+json` | MCP Server |
| `application/a2a-agent-card+json` | A2A Agent |
| `application/ai-skill` | AI Skill |
| `application/ai-catalog+json` | Nested sub-catalog |
| `application/ai-registry+json` | Searchable dynamic Registry |
| `application/vnd.oai.openapi+json` | OpenAPI description |

---

## 🔍 Discovery Status

Accessibility scan of ARD founding organizations' `/.well-known/ai-catalog.json` (as of 2026-06-29):

| Organization | Domain | Status | Notes |
|-------------|--------|--------|-------|
| ⚠️ Hugging Face | huggingface.co | **Published** | 超时，当前扫描环境无法连接 |
| ⚠️ DataRobot | datarobot.com | **Published** | 返回 403 (Varnish 防火墙拦截) |
| ✅ Suganthan | suganthan.com | **Published** | Individual developer, 3 entries |
| ✅ Joost de Valk | joost.blog | **Published** | Individual developer, 2 entries (MCP + A2A) |
| ⬜ Google | google.com | 404 | Not published |
| ⬜ Google Cloud | cloud.google.com | 404 | Not published |
| ⬜ Microsoft | microsoft.com | 404 | Not published |
| ⬜ GitHub | github.com | 404 | Not published (Agent Finder in Copilot) |
| ⬜ Snowflake | snowflake.com | 404 | Not published |
| ⬜ Salesforce | salesforce.com | No response | Not published |
| ⬜ Databricks | databricks.com | 404 | Not published |
| ⬜ NVIDIA | nvidia.com | 403 | Not publicly accessible |
| ⬜ GoDaddy | godaddy.com | 403 | Not publicly accessible |
| ⬜ ServiceNow | servicenow.com | 403 | Not publicly accessible |
| ⬜ Cisco | cisco.com | 404 | Not published |
| ⬜ Anthropic | anthropic.com | No response | Not a founding member |
| ⬜ OpenAI | openai.com | No response | Not a founding member |

> As of now, **only Hugging Face, DataRobot, Suganthan, and Joost de Valk** have published public `ai-catalog.json` files. However, as of 2026-06-30, Hugging Face is unreachable from the scan environment and DataRobot returns a 403. The ARD ecosystem is still in its early stages.

---

## 🚀 Quick Start

### Validate a Catalog

Use the official ARD JSON Schema:

```bash
npx ajv-cli validate -s spec/schemas/ai-catalog.schema.json -d catalogs/huggingface/ai-catalog.json
```

### Search with hf-discover

```bash
# Install hf-discover
pip install hf-discover

# Search for MCP servers in the Hugging Face ecosystem
hf discover search "transcribe some audio" --kind mcp

# Navigate to any website and discover its ARD catalog
hf discover navigate suganthan.com "find an MCP server for blog search"
```

---

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines.

If you find a new public `ai-catalog.json`, please:
1. Fork this repository
2. Add the catalog file under `catalogs/<publisher-name>/`
3. Update `INDEX.md`
4. Submit a Pull Request

### Guidelines
- Only accept **publicly accessible** `ai-catalog.json` files (no authentication required)
- Preserve the **original JSON format** without modification
- Include a `meta.json` file recording the source URL, collection date, and notes
- If a catalog goes offline or changes, update its status

---

## 📚 References

- [ARD Specification](https://github.com/ards-project/ard-spec) — The official ARD protocol spec
- [ARD JSON Schema](spec/schemas/ai-catalog.schema.json) — ai-catalog.json JSON Schema
- [ai-catalog Base Spec](https://github.com/Agent-Card/ai-catalog) — Underlying catalog format
- [Hugging Face Discover](https://github.com/huggingface/hf-discover) — HF ARD client/server
- [ARD Tutorial Lab](https://github.com/Agent-Field/agentic-resource-discovery-lab) — Hands-on ARD tutorial
- [Hugging Face Launch Blog](https://huggingface.co/blog/agentic-resource-discovery-launch) — ARD announcement post
- [DataRobot Integration Blog](https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/) — DataRobot ARD blog
- [Suganthan's ARD Blog](https://suganthan.com/blog/agentic-resource-discovery/) — Individual developer ARD experience
- [Joost's OKF + ARD Blog](https://joost.blog/okf-ard/) — Joost de Valk on combining OKF and ARD

---

## ⭐ Support

If you find this project useful, please consider giving it a star on GitHub — it helps others discover the ARD ecosystem!

[![Star History Chart](https://api.star-history.com/svg?repos=huangxn29/ard-catalogs&type=Date)](https://star-history.com/#huangxn29/ard-catalogs&Date)

---

## 📄 License

This repository is licensed under Apache 2.0. The collected `ai-catalog.json` files are copyright of their respective publishers and are included for educational/reference purposes only.
