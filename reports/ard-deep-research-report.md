# ARD (Agentic Resource Discovery) 协议深度调研报告

> 版本: 1.0 | 更新日期: 2026-06-26 | 状态: 定稿
> 作者: Research Agent | 领域: Agent 发现协议 / AI 互操作性

---

## 目录

1. [执行摘要](#1-执行摘要)
2. [背景与动机](#2-背景与动机)
3. [协议技术深度解析](#3-协议技术深度解析)
4. [生态全景分析](#4-生态全景分析)
5. [案例深度研究](#5-案例深度研究)
6. [竞争格局与对比分析](#6-竞争格局与对比分析)
7. [采用策略与落地建议](#7-采用策略与落地建议)
8. [挑战与风险](#8-挑战与风险)
9. [未来展望](#9-未来展望)
10. [附录](#10-附录)

---

## 1. 执行摘要

**Agentic Resource Discovery (ARD)** 是由 Linux Foundation AI Catalog Working Group 托管的开放、联合、去中心化的 Agent 资源发现协议。协议于 **2026 年 6 月 17 日** 发布 v0.9 草案，获得了 Google、Microsoft、Hugging Face、GitHub、NVIDIA、Snowflake、Salesforce、Cisco、Databricks、GoDaddy、ServiceNow 等业界巨头的联合支持。

ARD 的核心价值在于：**为 AI Agent 提供一种标准化的运行时发现机制**，使 Agent 能够在运行时动态搜索和发现所需的工具、MCP 服务器、A2A Agent 和 API，无需提前硬编码或预安装。

| 关键属性 | 值 |
|---------|-----|
| 协议版本 | v0.9 (Draft) |
| 托管组织 | Linux Foundation (AI Catalog Working Group) |
| 核心文件 | `/.well-known/ai-catalog.json` |
| 基本 URI 方案 | `urn:air:` |
| 当前采用状态 | 早期 — 4 个已确认公开部署 |
| 生态成熟度 | ⭐⭐ (2/5) — 刚起步，增长潜力大 |

**本报告结论：** ARD 代表了 AI Agent 互操作性领域的一个关键基础设施层。虽然目前处于极早期阶段（仅 4 个公开部署），但其强大的行业支持、扎实的技术设计和清晰的标准化路径使其成为值得密切跟踪和早期投入的战略性协议。

---

## 2. 背景与动机

### 2.1 为什么需要 ARD？

AI Agent 生态正经历爆炸式增长。Agent 需要调用各种外部资源来完成复杂任务——MCP 服务器、A2A Agent、REST API、数据集、知识库等。但当前生态面临几个关键痛点：

1. **发现困难**：Agent 如何知道存在哪些可用资源？缺乏统一的发现机制。
2. **耦合过紧**：Agent 通常需要提前硬编码资源地址，缺乏运行时动态发现能力。
3. **格式碎片化**：不同提供商使用不同的描述格式和注册方式。
4. **规模瓶颈**：LLM context window 无法容纳大量资源选择，需要可扩展的外部搜索机制。

### 2.2 解决思路

ARD 的设计团队（跨 Google、Microsoft、Hugging Face 等公司的专家）提出了一个三层的解决方案：

```
┌─────────────────────────────────────────────────────┐
│                     AI Agent                          │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │ MCP Client  │  │ A2A Client   │  │ HTTP Client │ │
│  └──────┬──────┘  └──────┬───────┘  └──────┬──────┘ │
└─────────┼────────────────┼──────────────────┼────────┘
          │                │                  │
┌─────────▼────────────────▼──────────────────▼────────┐
│              ARD Discovery Layer                       │
│  ┌──────────────────────────────────────────────────┐ │
│  │  Search / Crawl /.well-known/ai-catalog.json     │ │
│  └──────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────┘
          │                │                  │
┌─────────▼────────────────▼──────────────────▼────────┐
│         ARD Catalog 生态（ai-catalog.json）            │
│  ┌────────┐ ┌───────────┐ ┌─────────┐ ┌────────────┐ │
│  │MCP Srv │ │ A2A Agent │ │  Skills │ │  Registry  │ │
│  └────────┘ └───────────┘ └─────────┘ └────────────┘ │
└────────────────────────────────────────────────────────┘
```

### 2.3 与现有生态的关系

ARD 并非替代现有协议，而是作为它们的"发现层"：

| 协议 | 角色 | 与 ARD 的关系 |
|------|------|--------------|
| **MCP** (Model Context Protocol) | Agent ↔ 工具的标准接口 | ARD 可发现和索引 MCP Server |
| **A2A** (Agent-to-Agent) | Agent ↔ Agent 通信协议 | ARD 可发现和索引 A2A Agent |
| **OpenAPI** | REST API 描述规范 | ARD 可发现和索引 OpenAPI 端点 |
| **ARD** (本协议) | 以上所有资源的**发现层** | 为其他协议提供统一入口 |

---

## 3. 协议技术深度解析

### 3.1 架构总览

ARD 协议定义了三个核心概念：

```
┌──────────────────────────────────────────────────┐
│               ARD 三层架构                          │
├──────────────────────────────────────────────────┤
│                                                    │
│  1. Catalog（目录）                                 │
│     静态清单文件，发布在 `/.well-known/ai-catalog.json`│
│     包含一个或多个条目，可直接嵌入或引用。              │
│                                                    │
│  2. Registry（注册中心）                             │
│     支持搜索的动态注册中心，暴露 HTTP REST 搜索接口。  │
│     适用于大型生态（如 Hugging Face 的 Discover）。   │
│                                                    │
│  3. Agent（AI Agent 消费者）                         │
│     消费 Catalog/Registry 来发现所需资源的 AI Agent。 │
└──────────────────────────────────────────────────┘
```

### 3.2 ai-catalog.json 规范详解

#### 3.2.1 顶层结构

```json
{
  "specVersion": "1.0",
  "host": {
    "displayName": "发布者名称",
    "identifier": "did:web:example.com"
  },
  "entries": [
    // ... 零个或多个条目
  ]
}
```

| 字段 | 是否必须 | 说明 |
|------|---------|------|
| `specVersion` | 是 | 当前固定为 `"1.0"` |
| `host.displayName` | 是 | 发布者的可读名称 |
| `host.identifier` | 否 | 发布者的标准标识符（建议使用 `did:web`） |
| `entries` | 否 | 资源条目列表（可以为空，用于占位） |

#### 3.2.2 条目结构

每个 `entry` 包含以下字段：

```json
{
  "identifier": "urn:air:publisher:namespace:name",
  "displayName": "资源显示名称",
  "description": "资源描述",
  "type": "application/mcp-server-card+json",
  "url": "https://...",
  "data": { /* 内嵌数据，与 url 二选一或都提供 */ },
  "capabilities": ["能力1", "能力2"],
  "representativeQueries": ["示例查询1", "示例查询2"],
  "tags": ["标签1", "标签2"],
  "version": "1.0.0",
  "trustManifest": { /* 信任清单 */ }
}
```

| 字段 | 是否必须 | 说明 |
|------|---------|------|
| `identifier` | 是 | 全局唯一标识符，`urn:air:` 前缀 |
| `type` | 是 | IANA Media Type，标识资源类型 |
| `url` 或 `data` | 是（至少其一） | 远程引用或内嵌数据 |
| `displayName` | 否 | 但强烈建议提供 |
| `description` | 否 | 但强烈建议提供 |
| `capabilities` | 否 | 能力列表，辅助 Agent 匹配 |
| `representativeQueries` | 否 | 示例自然语言查询 |
| `tags` | 否 | 标签分类 |
| `version` | 否 | 语义版本号 |
| `trustManifest` | 否 | 信任和安全声明 |

#### 3.2.3 资源类型（Media Type）体系

| Media Type | 含义 | 典型用途 |
|-----------|------|---------|
| `application/mcp-server-card+json` | MCP 服务器 | 工具调用接口 |
| `application/a2a-agent-card+json` | A2A Agent | Agent 间通信 |
| `application/ai-skill` | AI Skill | 可复用的 AI 能力单元 |
| `application/ai-catalog+json` | 子 Catalog | 嵌套目录结构 |
| `application/ai-registry+json` | 动态注册中心 | 大规模可搜索注册中心 |
| `application/vnd.oai.openapi+json` | OpenAPI 描述 | REST API 接口 |
| `application/parquet` | 数据集 | 结构化数据 |
| `*/*` | 兜底类型 | 二进制文件或未知类型 |

> **设计亮点**: 使用 IANA Media Type 作为类型标识天然支持可扩展性。新增资源类型只需注册新的 Media Type，无需修改协议本身。

### 3.3 URN 命名规范

ARD 使用 `urn:air:` 作为基本 URI 方案，格式如下：

```
urn:air:<publisher>:<namespace>:<resource-name>
```

| 段 | 示例 | 说明 |
|---|------|------|
| `urn:air` | 固定前缀 | `air` = Agentic Resource Discovery |
| `<publisher>` | `huggingface.co` | 发布者的域名或唯一标识 |
| `<namespace>` | `registry`, `agent`, `server`, `api` | 资源类别命名空间 |
| `<resource-name>` | `discover`, `assistant`, `weather` | 资源的具体名称 |

**实际示例：**
- `urn:ai:huggingface.co:registry:discover`
- `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:model-training`
- `urn:ai:suganthan.com:agent:blog-agent`

### 3.4 搜索协议（Registry Search API）

Registry 资源类型要求暴露 HTTP REST 搜索接口：

```
GET /search?q={natural language query}&types={filter types}&limit={max results}
```

| 参数 | 是否必须 | 说明 |
|------|---------|------|
| `q` | 否 | 自然语言查询字符串 |
| `types` | 否 | 按 Media Type 过滤（逗号分隔） |
| `limit` | 否 | 最大返回结果数 |
| `offset` | 否 | 分页偏移量 |

**响应格式：**

```json
{
  "results": [
    {
      "identifier": "...",
      "displayName": "...",
      "description": "...",
      "type": "...",
      "url": "...",
      "score": 0.95,
      "reason": "该 MCP Server 与用户查询高度匹配"
    }
  ]
}
```

### 3.5 Value-or-Reference 设计

ARD 的关键原则之一是 **Value-or-Reference**：

| 模式 | 方式 | 适用场景 | 示例 |
|------|------|---------|------|
| **Reference** | `url` 指向远程 JSON | 资源频繁更新 / 大型条目 | MCP Server 配置 |
| **Value (inline)** | `data` 内嵌完整数据 | 静态配置 / 小型条目 | 嵌套子 Catalog |
| **Both** | 同时提供 url 和 data | 元数据 + 详细配置 | Catalog 入口汇总 |

### 3.6 信任与安全

ARD 通过 `trustManifest` 字段提供信任声明机制：

```json
"trustManifest": {
  "identity": "did:web:example.com",
  "identityType": "did:web",
  "attestations": [
    {
      "type": "JWS",
      "uri": "https://example.com/.well-known/jwks.json"
    }
  ]
}
```

| 字段 | 说明 |
|------|------|
| `identity` | 身份标识符 |
| `identityType` | 身份类型（`did:web`, `spiffe` 等） |
| `attestations[]` | 认证声明列表 |
| `attestations[].type` | 认证类型（`JWS`, `SPIFFE-X509`, `SOC2-Type2` 等） |
| `attestations[].uri` | 认证验证 URI |

**当前已知的信任模式：**
- `did:web` + JWS 签名（Joost de Valk 采用）
- `did:web` 标识（Suganthan 采用）
- SPIFFE X509 + SOC2 Type2（企业示例）

---

## 4. 生态全景分析

### 4.1 支持者图谱

```
          ┌──────────────────────────────────────┐
          │     AI Catalog Working Group         │
          │     (Linux Foundation 托管)            │
          └──────┬───────────────────┬───────────┘
                 │                   │
     ┌───────────▼──────┐    ┌──────▼────────────┐
     │  创始支持公司      │    │  社区参与者         │
     │  (v0.9 联合发布)   │    │  (已公开部署)       │
     ├───────────────────┤    ├──────────────────┤
     │ • Google          │    │ • Hugging Face   │
     │ • Microsoft       │    │ • DataRobot      │
     │ • Hugging Face    │    │ • Suganthan(个人) │
     │ • GitHub          │    │ • Joost(个人)     │
     │ • NVIDIA          │    │                   │
     │ • Snowflake       │    │                   │
     │ • Salesforce      │    │                   │
     │ • Cisco           │    │                   │
     │ • Databricks      │    │                   │
     │ • GoDaddy         │    │                   │
     │ • ServiceNow      │    │                   │
     └───────────────────┘    └──────────────────┘
```

### 4.2 采用阶段分析

```
                           公开 Catalog
                           已发布
              ┌─────────────────┬─────────────────┐
              │                 │                 │
          🏢 企业级          👤 个人开发者      ⚪ 尚未部署
              │                 │                 │
     ┌────────▼────┐   ┌───────▼───────┐   ┌────▼────────────┐
     │ DataRobot   │   │ Suganthan     │   │ Google          │
     │ Hugging Face│   │ Joost de Valk │   │ Microsoft       │
     └─────────────┘   └───────────────┘   │ GitHub          │
                                           │ NVIDIA          │
        已部署 2 家企业         已部署 2 位个人 │ Snowflake       │
        10 skills + 1 registry  5 个条目      │ Salesforce      │
                                           │ Databricks      │
                                           │ Cisco           │
                                           │ GoDaddy         │
                                           │ ServiceNow      │
                                           └─────────────────┘
                                           13 家创始成员尚未公开部署
```

### 4.3 当前 Catalog 覆盖统计

| 指标 | 数值 |
|------|------|
| 已收录发布者总数 | 4 |
| 总条目数 | 16 |
| Registry 条目 | 1 (Hugging Face) |
| Skill 条目 | 10 (DataRobot) |
| A2A Agent 条目 | 2 |
| MCP Server 条目 | 2 |
| OpenAPI 条目 | 1 |
| 已采用 did:web 身份 | 2 (Suganthan, Joost) |
| 已采用 trustManifest | 1 (Joost) |
| 已验证可访问率 | 100% (4/4) |

### 4.4 首发媒体与社区反响

**首批报道/博客文章（截至 2026-06-26）：**

| 日期 | 发布者 | 标题/主题 | 链接 |
|------|--------|-----------|------|
| 2026-06-17 | Hugging Face | "Agentic Resource Discovery Launch" | [博文](https://huggingface.co/blog/agentic-resource-discovery-launch) |
| 2026-06-17 | DataRobot | "DataRobot Agent Skills Discoverable through ARD" | [博文](https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/) |
| 2026-06-17 | Suganthan | "Agentic Resource Discovery — A hands-on experience" | [博文](https://suganthan.com/blog/agentic-resource-discovery/) |
| 2026-06-17 | Joost de Valk | "OKF & ARD" | [博文](https://joost.blog/okf-ard/) |

---

## 5. 案例深度研究

### 5.1 Hugging Face — 规模最大的 ARD 部署

**概况：** Hugging Face 作为最大的 AI 模型/工具托管平台，选择了 Registry 模式来索引其海量生态。

**技术选型分析：**

```
Hugging Face ARD 实现架构

┌─────────────────────────────────────────────────┐
│          huggingface.co/.well-known/             │
│             ai-catalog.json                      │
│  ┌─────────────────────────────────────────────┐ │
│  │ type: application/ai-registry+json          │ │
│  │ url: https://huggingface-hf-discover.hf.space│ │
│  └──────────────┬──────────────────────────────┘ │
└─────────────────┼────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│          hf-discover 注册中心服务                   │
│  (开源: https://github.com/huggingface/hf-discover)│
│                                                   │
│  ┌─────────────┐  ┌──────────┐  ┌──────────────┐ │
│  │ Skills 搜索  │  │ Spaces  │  │ MCP 服务器搜索│ │
│  └─────────────┘  └──────────┘  └──────────────┘ │
└─────────────────────────────────────────────────┘
```

**关键决策：** 选择 Registry 而非静态 Catalog 是因为 Hugging Face 的生态规模（数百万模型、Spaces、Skills），静态 JSON 无法有效管理。

**提供的 CLI 工具：** `hf-discover` — 开源 Python CLI
```bash
# 搜索功能
hf discover search "transcribe some audio" --kind mcp

# 导航发现
hf discover navigate suganthan.com "find an MCP server for blog search"
```

### 5.2 DataRobot — 最完整的 Skill 部署

**概况：** DataRobot 发布了 10 个 Agent Skills，覆盖 MLOps 全生命周期。

**发布的 Skills 清单：**

| # | Skill 名称 | 功能领域 |
|---|-----------|---------|
| 1 | Model Training | 模型训练、项目管理、AutoML |
| 2 | Model Deployment | 模型部署、部署管理 |
| 3 | Predictions | 预测推理、批量评分 |
| 4 | Feature Engineering | 特征工程、特征发现 |
| 5 | Model Monitoring | 模型性能监控、数据漂移检测 |
| 6 | Model Explainability | 预测解释、SHAP 值分析 |
| 7 | Data Preparation | 数据集上传、数据资产管理 |
| 8 | App Framework CI/CD | 应用模板的 CI/CD 流水线 |
| 9 | External Agent Monitoring | OpenTelemetry Agent 可观测性 |
| 10 | Agent Assist | AI Agent 构建与部署 |

**启示：** DataRobot 的部署展示了如何将现有平台能力通过 ARD 协议暴露给 AI Agent 生态。这是 ISV（独立软件供应商）的最佳参考案例。

### 5.3 Suganthan — 个人开发者的最佳实践

**概况：** 个人开发者 Suganthan 部署了一个包含 3 个条目的完整 ARD catalog，涵盖 A2A Agent、MCP Server 和 OpenAPI。

**值得关注的高级特性使用：**

1. **`representativeQueries`**：为每个条目提供自然语言示例查询，帮助 Agent 理解何时使用该资源
2. **`capabilities`**：明确声明能力集合，支持基于能力的匹配
3. **`version`**：使用语义化版本号，支持版本管理
4. **`did:web` 身份标识**：使用去中心化身份标识

**最佳实践要点：**
- 小型部署（<10 条目）适合静态 Catalog 模式
- 提供 `representativeQueries` 显著提升 Agent 的匹配准确率
- `did:web` 是当前最易实施的去中心化身份方案

### 5.4 Joost de Valk — 信任机制的先锋

**概况：** Yoast 创始人 Joost de Valk 发布了一个包含 MCP Server 和 A2A Agent 的 ARD Catalog。

**独特之处：**

1. **`did:web` + JWS 签名的 `trustManifest`** — 目前唯一实现信任声明的公开部署
2. **覆盖领域内容** — SEO、WordPress、AI、开放网络等专业领域知识
3. **双重暴露** — 同时提供 MCP Server（工具调用）和 A2A Agent（自然语言问答）

**信任模型：**

```
Joost 的信任链:
┌───────────────────────────────────────────────────┐
│  joost.blog/.well-known/ai-catalog.json           │
│  ┌───────────────────────────────────────────────┐│
│  │ trustManifest:                                ││
│  │   identity: did:web:joost.blog                ││
│  │   identityType: did:web                       ││
│  │   attestations:                               ││
│  │     - type: JWS                               ││
│  │       uri: https://joost.blog/.well-known/    ││
│  │           jwks.json                           ││
│  └───────────────────────────────────────────────┘│
└───────────────────────────────────────────────────┘
```

---

## 6. 竞争格局与对比分析

### 6.1 协议生态全景图

```
                    Agent 互操作性协议生态
                          2026

┌──────────────────────────────────────────────────────────┐
│                    发现层 (Discovery)                       │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  ARD (Agentic Resource Discovery) ★ 本报告主题        │ │
│  │  地位: 正在成为 Agent 资源发现的行业标准                │ │
│  │  支持: Google, MS, HF, NVIDIA, GitHub 等              │ │
│  └──────────────────────────────────────────────────────┘ │
├──────────────────────────────────────────────────────────┤
│                  连接层 (Connectivity)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐│
│  │ MCP ★        │  │ A2A ★        │  │ OpenAI           ││
│  │ (Anthropic)  │  │ (Google)     │  │ Function Calling ││
│  │ Tool ↔ Agent │  │ Agent↔Agent  │  │ 原生函数调用      ││
│  │ 1250+ Server │  │ 企业采用增长  │  │ 闭源生态          ││
│  └──────────────┘  └──────────────┘  └──────────────────┘│
├──────────────────────────────────────────────────────────┤
│                  描述层 (Description)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐│
│  │ OpenAPI      │  │ JSON Schema  │  │ gRPC / Protobuf ││
│  │ REST API描述  │  │ 数据验证规范  │  │ RPC 框架         ││
│  │ 最广泛采用   │  │ 通用性最强   │  │ 高性能场景       ││
│  └──────────────┘  └──────────────┘  └──────────────────┘│
└──────────────────────────────────────────────────────────┘
```

### 6.2 横向对比

| 维度 | ARD | MCP | A2A | OpenAPI | Function Calling |
|------|-----|-----|-----|---------|-----------------|
| **定位** | 发现层 | 工具调用 | Agent 通信 | API 描述 | 函数调用 |
| **标准化** | Linux Foundation | Anthropic 主导 | Google 主导 | OpenAPI Initiative | 厂商私有 |
| **核心格式** | ai-catalog.json | JSON-RPC | JSON-RPC | OpenAPI Spec | JSON Schema |
| **开放程度** | ✅ 完全开放 | ✅ 开放 | ✅ 开放 | ✅ 开放 | ❌ 绑定 LLM 厂商 |
| **发现能力** | ★★★★★ | ★★☆☆☆ | ★☆☆☆☆ | ★★☆☆☆ | ★☆☆☆☆ |
| **运行时调用** | ★☆☆☆☆ (不做) | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| **信任/安全** | 内置 trustManifest | 传输层 | 传输层 | OAuth/OIDC | 传输层 |
| **当前采用** | 4 部署 (~10家支持) | 1250+ Server | 企业早期 | 数百万 API | OpenAI 生态 |
| **分层设计** | 清晰分层 | 单体 | 单体 | 单层 | 单体 |
| **新增类型扩展** | 通过新 Media Type | 扩展协议 | 扩展协议 | 扩展 spec | 厂商决定 |

### 6.3 互补而非竞争

ARD 与 MCP、A2A 的关系不是竞争，而是互补：

```
一个 Agent 使用所有协议的典型流程:

1. 发现 ──── ARD ──── 通过 ai-catalog.json 搜索可用资源
                   │
2. 选择 ──── Agent 根据任务选择合适的资源
                   │
3. 连接 ──── MCP (工具) / A2A (Agent) / HTTP (API)
                   │
4. 执行 ──── 完成任务
```

### 6.4 ARD 的独特优势

| 优势 | 说明 | 竞争对手难以复制的点 |
|------|------|-------------------|
| **行业共识** | 11 家巨头联合支持，Linux Foundation 托管 | 单一厂商协议无法形成行业共识 |
| **协议无关** | 不绑定任何连接协议 (MCP/A2A/OpenAPI) | 可成为"协议之上的协议" |
| **去中心化** | 无中心注册机构，只需在自有域名发布 | Scalability 远超中心化方案 |
| **渐进采用** | 从静态 JSON 到动态 Registry，逐步升级 | 适应从小到大的各种规模 |
| **安全设计** | trustManifest + did:web 原生支持 | 行业级别的信任框架 |
| **搜索优先** | 将选择从 LLM Context Window 中解放出来 | 解决 Agent 规模化的核心瓶颈 |

---

## 7. 采用策略与落地建议

### 7.1 采用路线图（分阶段）

```
阶段 1: 基础部署 (1-2 周)
┌─────────────────────────────────────────────────────┐
│  ✅ 发布 `/.well-known/ai-catalog.json`               │
│  ✅ 注册条目，指向已有的 MCP Server / API              │
│  ✅ 验证 JSON 合法性（使用官方 Schema）                │
│  ✅ 加入本仓库（ard-catalogs）索引                     │
└─────────────────────────────────────────────────────┘
                       │
                       ▼
阶段 2: 增强优化 (2-4 周)
┌─────────────────────────────────────────────────────┐
│  ✅ 添加 representativeQueries 提升匹配率             │
│  ✅ 实现 did:web 身份标识                            │
│  ✅ 添加 trustManifest + JWS 签名                    │
│  ✅ 添加 capabilities 和 tags 丰富元数据              │
└─────────────────────────────────────────────────────┘
                       │
                       ▼
阶段 3: 规模扩展 (1-3 个月)
┌─────────────────────────────────────────────────────┐
│  ⬜ 实现 Registry API 搜索服务（资源 >50 时必需）      │
│  ⬜ 集成 hf-discover 等 ARD 客户端工具                 │
│  ⬜ 监控 ARD 调用分析和优化                            │
│  ⬜ 参与 AI Catalog Working Group 标准演进             │
└─────────────────────────────────────────────────────┘
```

### 7.2 不同角色的采用建议

#### 🏢 企业/平台方（如 DataRobot）

**核心目标：** 将自己的平台能力暴露给 AI Agent 生态

- **优先级：高** — 这是增量获客的新渠道
- **建议方式：** 发布 Skill 条目，覆盖平台核心 API
- **参考案例：** DataRobot 的 10 个 Agent Skills
- **预期收益：** 被 Agent 开发者发现 → 试用 → 付费转化
- **投入估算：** 1-2 人周完成初始部署

#### 👤 个人开发者/创作者（如 Suganthan, Joost）

**核心目标：** 将自己的创作内容通过 Agent 触达更多用户

- **优先级：中高** — 内容分发的新渠道
- **建议方式：** 发布 MCP Server + A2A Agent 条目
- **参考案例：** Suganthan 的博客 Agent + MCP
- **预期收益：** 内容被 Agent 发现和引用
- **投入估算：** 数小时即可完成

#### 🛠️ 工具/框架开发者

**核心目标：** 为自己的开源工具或框架增加 ARD 支持

- **优先级：中** — 趋势布局，抢占生态位
- **建议方式：** 开源 MCP Server + ARD Catalog
- **参考案例：** Hugging Face 的 hf-discover
- **预期收益：** 成为 ARD 生态的基础设施
- **投入估算：** 取决于工具复杂度

### 7.3 具体实施检查清单

**基础部署清单：**

- [ ] 阅读并理解 ARD v0.9 规范
- [ ] 创建 `/.well-known/ai-catalog.json` 文件
- [ ] 设置正确的 `Content-Type` 响应头
- [ ] 使用官方 JSON Schema 验证文件合法性
- [ ] 至少注册一个有效条目
- [ ] 测试 Agent 能否成功发现和解析

**增强优化清单：**

- [ ] 为每个条目编写自然的 `representativeQueries`
- [ ] 声明 `capabilities` 数组
- [ ] 使用 `did:web` 格式的 `host.identifier`
- [ ] 生成并配置 JWS 签名的 `trustManifest`
- [ ] 添加 `tags` 并保持一致性
- [ ] 维护 `version` 语义版本号
- [ ] 在 ardcatalogs 仓库提交收录

### 7.4 风险与缓解

| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|---------|
| 协议仍在 Draft 阶段可能变更 | 高 | 中 | 跟踪 Working Group 动态，使用语义版本管理 |
| 生态早期，Agent 消费者少 | 中 | 中高 | 先占位，Early Mover Advantage |
| 信任/安全问题 | 中 | 高 | 尽早实现 did:web + JWS 签名 |
| 其他发现协议出现 | 低 | 高 | ARD 的行业共识是最强护城河 |
| Media Type 注册流程不确定性 | 低 | 中 | 目前支持的 6 种已覆盖主要场景 |

---

## 8. 挑战与风险

### 8.1 技术挑战

| 挑战 | 严重程度 | 说明 |
|------|---------|------|
| **Draft 阶段不稳定性** | 🟡 中 | v0.9 草案，规范可能变更，需要跟踪 WG |
| **搜索质量** | 🟡 中 | Registry 搜索结果的准确性依赖底层实现 |
| **大规模性能** | 🟢 低 | 静态 Catalog 无性能问题，Registry 需自行保障 |
| **Media Type 注册** | 🟢 低 | 已有标准流程，目前 6 种类型已够用 |
| **跨域发现** | 🟡 中 | Agent 如何知道搜索哪些域名的 Catalog？ |

### 8.2 生态挑战

| 挑战 | 严重程度 | 说明 |
|------|---------|------|
| **冷启动问题** | 🔴 高 | 需要足够多的 Catalog 发布者和 Agent 消费者同时存在 |
| **巨头采用缓慢** | 🔴 高 | 13 家创始成员中仅 2 家实际部署 |
| **Agent 框架集成** | 🟡 中 | 主流 Agent 框架需要原生支持 ARD 发现 |
| **开发者教育** | 🟡 中 | 需要更多教程、工具链降低门槛 |
| **与闭源生态竞争** | 🟢 低 | OpenAI Function Calling 等闭源方案各有其封闭性局限 |

### 8.3 治理挑战

| 挑战 | 严重程度 | 说明 |
|------|---------|------|
| **标准化进程** | 🟡 中 | Linux Foundation 托管，但标准化速度取决于工作组效率 |
| **Media Type 管控** | 🟢 低 | IANA 注册流程成熟，但新类型审批需时间 |
| **信任框架标准化** | 🟡 中 | trustManifest 的认证类型需要行业共识 |

---

## 9. 未来展望

### 9.1 短期 (3-6 个月)

| 预测 | 置信度 | 依据 |
|------|--------|------|
| ⭐ 至少有 5-10 家企业级 Catalog 发布 | 高 | 创始成员中多家在内部测试 |
| ⭐ Agent 框架（LangChain, CrewAI 等）集成 ARD | 中高 | 社区需求明确 |
| ⭐ 规范从 v0.9 推进到 v1.0 RC | 中 | 工作组目标明确 |
| ⭐ 出现更多 ARD 客户端工具（类似 hf-discover） | 中 | 开源社区响应积极 |

### 9.2 中期 (6-18 个月)

| 预测 | 置信度 | 依据 |
|------|--------|------|
| ⭐ MCP Server 默认附带 ARD Catalog 成为最佳实践 | 中高 | 已有社区讨论 |
| ⭐ ARD 成为 Agent 发现资源的"标准入口" | 中 | 行业共识正在形成 |
| ⭐ `/.well-known/ai-catalog.json` 普及率显著提升 | 中 | 类似 `robots.txt` / `.well-known` 的传播路径 |
| ⭐ 出现企业级 ARD Registry 服务（SaaS） | 中 | 市场需求明确 |
| ⭐ 与 DID/VC 信任体系深度集成 | 中 | 已有 Joost 的先例 |

### 9.3 长期 (18 个月+)

| 预测 | 置信度 | 依据 |
|------|--------|------|
| ⭐ ARD 成为 AI Agent 基础设施的"HTTP 级别的标准" | 中 | 类比 Web 的标准化历史 |
| ⭐ Agent 原生支持 ARD 发现（类似浏览器支持 DNS） | 中低 | 取决于 Agent 生态成熟度 |
| ⭐ 跨组织的 Agent 市场/经济体系基于 ARD | 低 | 愿景级别，但可信 |
| ⭐ ARD 与 Web 搜索引擎在 Agent 场景中融合 | 低 | 技术路径可能交汇 |

### 9.4 关键里程碑跟踪

| 里程碑 | 当前状态 | 意义 |
|--------|---------|------|
| ✅ v0.9 Draft 发布 | ✅ 已完成 (2026-06-17) | 协议公开 |
| ⬜ 首批 10 家企业 Catalog | 4/10 | 生态临界点 |
| ⬜ v1.0 正式发布 | 未开始 | 协议稳定 |
| ⬜ MCP SDK 原生 ARD 支持 | 未开始 | 开发者采纳加速 |
| ⬜ LangChain/CrewAI 集成 | 未开始 | Agent 框架支持 |
| ⬜ `/.well-known/ai-catalog.json` 普及率 >1% TOP1000 | 未开始 | 主流化指标 |

---

## 10. 附录

### 10.1 参考资源

#### 官方规范与仓库

| 资源 | 链接 |
|------|------|
| ARD 官方规范 | https://github.com/ards-project/ard-spec |
| ai-catalog 基础规范 | https://github.com/Agent-Card/ai-catalog |
| ARD v0.9 Draft | 参见 `spec/` 目录 |
| JSON Schema | `spec/schemas/ai-catalog.schema.json` |

#### 公开 Catalog 文件

| 发布者 | Catalog URL |
|--------|-------------|
| Hugging Face | https://huggingface.co/.well-known/ai-catalog.json |
| DataRobot | https://datarobot.com/.well-known/ai-catalog.json |
| Suganthan | https://suganthan.com/.well-known/ai-catalog.json |
| Joost de Valk | https://joost.blog/.well-known/ai-catalog.json |

#### 工具与实现

| 工具 | 链接 |
|------|------|
| hf-discover (Hugging Face ARD 客户端) | https://github.com/huggingface/hf-discover |
| ARD 教程实验室 | https://github.com/Agent-Field/agentic-resource-discovery-lab |
| ard-catalogs 收录仓库 | 本仓库 |

#### 博客与报道

| 标题 | 链接 |
|------|------|
| Hugging Face 发布博文 | https://huggingface.co/blog/agentic-resource-discovery-launch |
| DataRobot 集成博文 | https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/ |
| Suganthan 实现体验 | https://suganthan.com/blog/agentic-resource-discovery/ |
| Joost de Valk OKF & ARD | https://joost.blog/okf-ard/ |

### 10.2 术语表

| 术语 | 英文 | 定义 |
|------|------|------|
| Agent | AI Agent | 能够自主执行任务的 AI 程序 |
| ARD | Agentic Resource Discovery | 本协议，Agent 资源发现协议 |
| Catalog | Catalog | 机器可读的资源清单文件 (ai-catalog.json) |
| Registry | Registry | 支持搜索的动态注册中心 |
| MCP | Model Context Protocol | Anthropic 提出的 Agent-工具连接协议 |
| A2A | Agent-to-Agent | Google 提出的 Agent-Agent 通信协议 |
| Media Type | IANA Media Type | 用于标识资源类型的标准化字符串 |
| URN | Uniform Resource Name | 持久、位置无关的资源标识符 |
| DID | Decentralized Identifier | 去中心化身份标识符 |
| JWS | JSON Web Signature | JSON 签名格式，用于信任验证 |
| Working Group | AI Catalog Working Group | Linux Foundation 下的 ARD 标准工作组 |
| Skill | AI Skill | 可复用的 AI 能力单元 |
| trustManifest | Trust Manifest | ARD 中的信任声明数据结构 |

### 10.3 分析方法说明

本报告采用以下研究方法：

1. **协议规范分析**: 直接研读 ARD v0.9 草案和 ai-catalog JSON Schema
2. **实证验证**: 对每个已知的公开 Catalog 进行 HTTP 可达性验证和内容分析
3. **横向对比**: 与 MCP、A2A、OpenAPI、Function Calling 等协议进行系统对比
4. **案例研究**: 对每个已部署案例进行深度的技术选型和架构分析
5. **生态扫描**: 跟踪 13 家创始支持者的公开部署状态

> **免责声明**: 本报告基于 2026-06-26 的公开信息。ARD 协议处于早期 Draft 阶段，规范内容可能发生变化。报告中的预测分析仅代表基于当前信息的合理推断，不构成投资或商业决策建议。

---

*报告结束。本报告由 ARD 技术研究员自动生成。*
