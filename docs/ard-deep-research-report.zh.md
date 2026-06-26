# 🧠 ARD（Agentic Resource Discovery）协议深度调研报告

> **版本**: v1.0 | **日期**: 2026-06-26 | **分类**: 技术调研 / 行业分析

---

## 目录

1. [摘要](#1-摘要)
2. [背景与起源](#2-背景与起源)
3. [协议架构与技术细节](#3-协议架构与技术细节)
4. [生态系统与当前采用状况](#4-生态系统与当前采用状况)
5. [行业竞品格局分析](#5-行业竞品格局分析)
6. [ARD 与 MCP、A2A 的协同关系](#6-ard-与-mcp-a2a-的协同关系)
7. [信任与安全模型](#7-信任与安全模型)
8. [挑战与局限性](#8-挑战与局限性)
9. [未来展望与路线图](#9-未来展望与路线图)
10. [落地建议](#10-落地建议)
11. [参考文献](#11-参考文献)

---

## 1. 摘要

**Agentic Resource Discovery（ARD）** 是由 **Google、Microsoft、Hugging Face、GitHub、NVIDIA、Snowflake、Salesforce、Cisco、Databricks、GoDaddy、ServiceNow** 等 11 家科技巨头联合推出的开放协议规范，于 **2026 年 6 月 17 日** 正式发布草案 v0.9，由 **Linux Foundation** 下的 **AI Catalog 工作组** 托管，采用 **Apache 2.0** 许可。

ARD 的核心目标是为 AI Agent 提供一套标准的**运行时资源发现机制**——让 Agent 能够像人类使用搜索引擎一样，通过自然语言查询动态发现所需工具、MCP 服务器、A2A Agent、API 等能力，而不再依赖预安装或硬编码的配置。

本报告从协议背景、技术架构、生态现状、竞品对比、信任模型、挑战与未来展望等多个维度，对 ARD 协议进行深度调研分析，并提供落地建议。

---

## 2. 背景与起源

### 2.1 时代背景：Agent 生态的"巴别塔"困境

2025-2026 年，AI Agent 领域经历了爆发式增长，但面临严重的**互操作性危机**：

- **MCP（Model Context Protocol）**：Anthropic 推出的工具连接协议，解决了 Agent 如何调用工具的标准化问题
- **A2A（Agent-to-Agent）**：Google 推出的 Agent 间通信协议，解决了 Agent 如何协作的标准化问题
- **但缺少关键一环**：Agent 如何**发现**可用的工具和其他 Agent？

现实情况是，开发者不得不手动配置每个 Agent 可用的工具列表。当工具数量从几个增长到成百上千时，这种模式完全不可持续。

### 2.2 ARD 的诞生

2026 年 6 月 17 日，Google 在官方开发者博客上正式宣布了 ARD 规范，立即获得 10 家行业巨头的背书。这一事件被多家媒体形容为"AI Agent 的搜索引擎诞生"。

**关键时间线：**

| 时间 | 事件 |
|------|------|
| 2024年11月 | Anthropic 发布 MCP 协议 |
| 2025年4月 | Google 发布 A2A 协议（50+合作伙伴） |
| 2025年7月 | Cisco 将 AGNTCY 项目捐赠给 Linux Foundation |
| 2026年6月17日 | ARD v0.9 草案正式发布 |
| 2026年6月 | Hugging Face 发布 hf-discover 工具 |
| 2026年6月 | DataRobot 发布 ARD 兼容的 Agent Skills 目录 |
| 2026年（未来数月） | Google Agent Registry 计划集成原生 ARD 支持 |

### 2.3 战略意义——行业格局解读

ARD 的推出被业界视为**传统企业软件巨头对 OpenAI 和 Anthropic 的生态围堵**。

- OpenAI 和 Anthropic 均**不在**初始支持者名单中
- ARD 的 11 家支持者覆盖了云计算（Google Cloud、Microsoft Azure）、开发者平台（GitHub）、AI 社区（Hugging Face）、企业软件（Salesforce、ServiceNow）、数据基础设施（Snowflake、Databricks）、硬件（NVIDIA）、网络基础设施（Cisco）、互联网服务（GoDaddy）
- 这些公司共同构建了一个**开放但联合**的生态联盟，旨在防止 Agent 生态被单一公司锁定

---

## 3. 协议架构与技术细节

### 3.1 核心设计原则

| 原则 | 说明 |
|------|------|
| **Search-First Discovery** | Agent 通过搜索动态发现，而非预安装 |
| **Scalability Beyond Context Windows** | 将选择逻辑从 LLM 上下文窗口移至专用搜索服务 |
| **Artifact Agnostic Envelope** | 使用 IANA Media Type 标识资源类型，不约束内部 Schema |
| **Value-or-Reference** | 条目必须包含 url（远程引用）或 data（内嵌）之一，不可兼得或全无 |
| **REST API Baseline** | 注册中心必须暴露 HTTP REST 搜索接口 |

### 3.2 两大核心原语

ARD 定义了两个互补的原语：

#### ① Catalog（目录）

组织在其域名的 `/.well-known/ai-catalog.json` 路径下发布一个静态清单文件，列出其提供的所有 AI 能力。

**Catalog 文件结构**（基于 JSON Schema）：

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
      "description": "全文搜索服务，支持语义检索和关键词匹配",
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

**关键字段说明：**

| 字段 | 必需 | 说明 |
|------|------|------|
| `specVersion` | ✅ | 当前必须为 `"1.0"`（注意：协议是 v0.9 草案但 Schema 版本为 1.0） |
| `host.displayName` | ✅ | 发布者的可读名称 |
| `host.identifier` | 否 | 可验证标识（如 `did:web:domain.com`） |
| `entries[].identifier` | ✅ | URN 格式的唯一标识（`urn:air:<publisher>:<namespace>:<name>`） |
| `entries[].type` | ✅ | IANA Media Type 标示资源类型 |
| `entries[].url` 或 `data` | ✅ 二选一 | 远程引用或内嵌数据 |
| `entries[].representativeQueries` | 否 | 2-5 条代表性搜索查询（用于向量索引和检索增强） |

#### ② Registry（注册中心）

注册中心是"Agent 的搜索引擎"——它爬取各个域名的 Catalog，构建索引，并通过 `POST /search` API 提供运行时搜索服务。

**Registry 搜索请求示例：**

```http
POST /search HTTP/1.1
Content-Type: application/json

{
  "query": "find an MCP server for database query",
  "type": "application/mcp-server-card+json",
  "limit": 10
}
```

**Registry 搜索响应示例：**

```json
{
  "results": [
    {
      "identifier": "urn:air:example.com:server:db-mcp",
      "displayName": "Database Query MCP Server",
      "type": "application/mcp-server-card+json",
      "url": "https://example.com/cards/db-mcp.json",
      "score": 0.92,
      "description": "自然语言数据库查询接口",
      "tags": ["database", "sql", "query"]
    }
  ],
  "totalResults": 1
}
```

### 3.3 支持的资源类型（IANA Media Types）

| Media Type | 含义 | 示例场景 |
|------------|------|----------|
| `application/mcp-server-card+json` | MCP 服务器 | 数据库查询工具、文件处理工具 |
| `application/a2a-agent-card+json` | A2A Agent | 客服 Agent、数据分析 Agent |
| `application/ai-skill` | AI Skill | 模型训练技能、数据预处理技能 |
| `application/ai-catalog+json` | 嵌套子 Catalog | 大型组织的分层目录 |
| `application/ai-registry+json` | 可搜索的动态注册中心 | Hugging Face Discover、GitHub Agent Finder |
| `application/vnd.oai.openapi+json` | OpenAPI 描述 | REST API 接口 |

### 3.4 URN 命名规范

ARD 使用 RFC 8141 兼容的 URN 格式：

```
urn:air:<publisher>:<namespace>:<agent-name>
```

其中：
- `air` = Agentic Resource 的固定命名空间标识
- `publisher` = 发布者域名或 DID（如 `huggingface.co`、`github.com:datarobot-oss`）
- `namespace` = 资源分类（如 `registry`、`server`、`agent`、`skill`、`api`）
- `agent-name` = 具体资源名称（如 `discover`、`blog-mcp`、`model-training`）

**实际示例：**

| URN | 归属 |
|-----|------|
| `urn:ai:huggingface.co:registry:discover` | Hugging Face Discover 注册中心 |
| `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:model-training` | DataRobot 模型训练 Skill |
| `urn:ai:suganthan.com:server:blog-mcp` | Suganthan 博客 MCP 服务器 |
| `urn:ai:joost.blog:a2a:ask-joost` | Joost de Valk 的 A2A Agent |

### 3.5 发现流程（完整链路）

```
Agent ──(1) 自然语言查询──▶ Registry
                            │
                      (2) 搜索索引
                            │
                            ▼
                      ┌─────────────┐
                      │ 索引数据库   │
                      │(向量+关键词) │
                      └─────────────┘
                            │
                      (3) 返回匹配结果
                            │
                            ▼
Agent ◀──(4) 带信任元数据的排序结果─── Registry
  │
  │ (5) 从结果中选择目标
  │ (6) 通过目标 URL 获取详细卡片
  │ (7) 验证发布者身份（did:web / JWS）
  ▼
Target MCP Server / A2A Agent / API
  │
  │ (8) 通过自身协议（MCP/A2A/REST）调用
  ▼
能力执行
```

---

## 4. 生态系统与当前采用状况

### 4.1 已发布公开 Catalog 的组织

截至 2026-06-26，已有 **4 个组织/个人** 发布了公开的 `ai-catalog.json`：

| # | 发布者 | 类型 | 条目数 | 资源类型 | 高级特性 |
|---|--------|------|--------|----------|----------|
| 1 | **Hugging Face** (huggingface.co) | 企业 | 1 (Registry) | `ai-registry` | 指向动态 Discover 注册中心 |
| 2 | **DataRobot** (datarobot.com) | 企业 | 10 | `ai-skill` | 覆盖 MLOps 全生命周期 |
| 3 | **Suganthan Mohanadasan** (suganthan.com) | 个人 | 3 | `a2a-agent`, `mcp-server`, `openapi` | `representativeQueries`, `capabilities`, `version`, `did:web` |
| 4 | **Joost de Valk** (joost.blog) | 个人 | 2 | `mcp-server`, `a2a-agent` | `did:web`, JWS 签名的 `trustManifest` |

### 4.2 已宣布支持但尚未发布 Catalog 的组织

| 组织 | 计划集成方式 | 预期时间 |
|------|-------------|----------|
| **Google** | Agent Registry（Gemini Enterprise Agent Platform 的一部分） | 未来数月 |
| **GitHub** | Copilot Agent Finder | 已部分上线 |
| **Cisco** | AGNTCY Agent Directory（作为 ARD 参考实现） | 已开源发布 |
| **Salesforce** | 原生 ARD 集成到 Salesforce Platform | TBD |
| **Snowflake** | 通过 CoWork 和 CoCo 实现发现 | TBD |

### 4.3 参考实现与工具

| 工具 | 开发者 | 说明 | 地址 |
|------|--------|------|------|
| **hf-discover** | Hugging Face | ARD 客户端 CLI + 服务端，支持搜索、导航、自建 Registry | [GitHub](https://github.com/huggingface/hf-discover) |
| **AGNTCY Directory** | Cisco / Linux Foundation | 基于 OCI、DHT 的分布式 Agent 目录服务 | [GitHub](https://github.com/agntcy) |
| **ard-spec** | ARD 工作组 | 官方规范仓库，含 JSON Schema、CDDL、URN 指南 | [GitHub](https://github.com/ards-project/ard-spec) |
| **ARD Lab** | Agent Field | 端到端实操教程（含跨组织数据分析示例） | [GitHub](https://github.com/Agent-Field/agentic-resource-discovery-lab) |
| **ARD 快速开始** | Google | 官方快速入门站点 | [agenticresourcediscovery.org](https://agenticresourcediscovery.org) |

### 4.4 可访问性扫描结果

对 11 家创始组织的 `/.well-known/ai-catalog.json` 扫描：

| 组织 | 域名 | HTTP 状态 | 说明 |
|------|------|-----------|------|
| ✅ Hugging Face | huggingface.co | 200 | 已发布 |
| ✅ DataRobot | datarobot.com | 200 | 已发布 |
| ✅ Suganthan | suganthan.com | 200 | 已发布 |
| ✅ Joost de Valk | joost.blog | 200 | 已发布 |
| ⬜ Google | google.com | 404 | 未发布 |
| ⬜ Google Cloud | cloud.google.com | 404 | 未发布 |
| ⬜ Microsoft | microsoft.com | 404 | 未发布 |
| ⬜ GitHub | github.com | 404 | 未发布 |
| ⬜ Snowflake | snowflake.com | 404 | 未发布 |
| ⬜ Salesforce | salesforce.com | 无响应 | 未发布 |
| ⬜ Databricks | databricks.com | 404 | 未发布 |
| ⬜ NVIDIA | nvidia.com | 403 | 未公开 |
| ⬜ GoDaddy | godaddy.com | 403 | 未公开 |
| ⬜ ServiceNow | servicenow.com | 403 | 未公开 |
| ⬜ Cisco | cisco.com | 待扫描 | 未扫描 |

> **结论**: ARD 生态处于**极早期阶段**。大多数创始组织尚未发布公开 Catalog，这既是挑战也是机遇——先行者将获得生态位优势。

---

## 5. 行业竞品格局分析

### 5.1 Agent 发现/注册方案全景对比

| 方案 | 类型 | 治理模型 | 信任模型 | 适用场景 |
|------|------|----------|----------|----------|
| **ARD (Catalogs + Registries)** | 开放标准 | Linux Foundation | 域名锚定 + did:web 可选 | 通用跨组织发现 |
| **MCP Registry** | 集中式 | GitHub | GitHub OAuth + DNS TXT | MCP 服务器发现 |
| **A2A Agent Cards** | 去中心化 | 无中央机构 | TLS + OAuth2 | Agent 间自描述 |
| **AGNTCY Directory** | 分布式 P2P | Linux Foundation | OCI 签名 + DHT | 企业级联邦目录 |
| **NANDA AgentFacts** | 去中心化可验证 | MIT AIDE | W3C VC 可验证凭证 | 隐私保护的凭证化发现 |
| **Microsoft Entra Agent ID** | 企业 SaaS | Microsoft | Azure AD / Zero Trust | 微软生态企业级管理 |
| **agent:// 协议** | URI 方案 | IETF（草案） | 视下层实现而定 | 通用 Agent 寻址 |

### 5.2 ARD 的独特定位

ARD 的差异化优势在于：

1. **联盟背书**：11 家行业巨头联合推出，非单一厂商控制
2. **域名即信任**：利用成熟 DNS 基础设施作为信任锚点，无需额外 PKI
3. **协议无关**：不取代 MCP/A2A，而是作为发现层与其协同
4. **Value-or-Reference 模式**：灵活支持内嵌数据或远程引用
5. **统一的搜索语义**：通过 `representativeQueries` 实现语义搜索

### 5.3 主要不足

1. **生态极早期**：大部分创始组织尚未实际部署
2. **缺乏生产级 Registry**：尚无大规模公开 Registry 在运行
3. **无强制信任机制**：`trustManifest` 是可选的，早期 Catalog 普遍未包含
4. **Schema 版本混乱**：协议版本 v0.9 但 Schema 字段要求 `"1.0"`
5. **OpenAI 和 Anthropic 缺席**：两大关键参与者未加入，可能导致标准分裂

---

## 6. ARD 与 MCP、A2A 的协同关系

### 6.1 三层协议栈模型

行业正在形成共识——ARD、MCP、A2A 不是竞争关系，而是 Agent 互操作性的**三个互补层**：

```
  ┌─────────────────────────────────┐
  │   A2A (Agent-to-Agent)          │  ← 水平协作：Agent 之间如何通信
  │   Google 主导                   │
  ├─────────────────────────────────┤
  │   ARD (Agentic Resource Disc.)  │  ← 发现层：在哪里找到能力
  │   Linux Foundation 托管         │
  ├─────────────────────────────────┤
  │   MCP (Model Context Protocol)  │  ← 工具接入：Agent 如何调用工具
  │   Anthropic 主导                │
  └─────────────────────────────────┘
```

### 6.2 类比理解：网络协议栈

| AI Agent 层 | 网络协议类比 | 说明 |
|-------------|-------------|------|
| **MCP** | 二层（数据链路） | 提供详细的工具级连接能力，扩展有限 |
| **ARD** | DNS + 搜索引擎 | 将"资源在哪里"的问题从上下文窗口解耦到搜索服务 |
| **A2A** | 三层（路由） | 汇聚能力边界，提供跨域的协作网关 |

### 6.3 三者协作的完整流程

以"企业智能助手自动完成报销流程"为例：

```
1. Agent 通过 A2A Agent Card 发现"财务助手"的存在
2. Agent 通过 ARD 搜索找到一个"报销单 MCP Server"
3. Agent 通过 MCP 协议连接并调用该 MCP Server
4. MCP Server 返回查询结果
5. Agent 通过 A2A 将结果传递给"审批 Agent"进行下一步处理
```

### 6.4 选择建议

| 场景 | 推荐方案 |
|------|----------|
| 单一 Agent 需要外部工具 | MCP 即可满足 |
| 多 Agent 系统需要编排 | A2A + MCP |
| 跨组织发现 Agent 和工具 | **ARD + MCP/A2A** |
| 企业级 Agent 目录 + 治理 | ARD (AGNTCY) + Microsoft Entra |
| 个人开发者发布个人 Agent | ARD Catalog（最简单的方式） |

---

## 7. 信任与安全模型

### 7.1 分层信任架构

ARD 的信任模型分为三层：

#### 第一层：域名锚定（基础）

- 发布者通过拥有域名来证明身份的基础信任
- Catalog 通过 HTTPS 提供，依赖 TLS 证书验证
- 解析路径固定为 `/.well-known/ai-catalog.json`

#### 第二层：`did:web` 身份标识（增强）

- 使用 `did:web:domain.com` 作为 `host.identifier`
- 无需区块链，直接利用 DNS 基础设施
- DID 文档发布在 `/.well-known/did.json`
- Suganthan 和 Joost de Valk 已采用此方案

#### 第三层：`trustManifest`（最高级别）

包含可选的加密安全信封，提供：

| 组件 | 说明 |
|------|------|
| `identity` | 全局唯一加密标识（SPIFFE ID 或 DID URI） |
| `identityType` | 格式提示（spiffe / did / https / other） |
| `trustSchema` | 信任框架标识和版本 |
| `attestations` | 可验证声明（SOC 2、HIPAA 等合规审计） |
| `provenance` | 加密谱系追踪（派生来源、发布时间） |
| `signature` | detached JWS 签名（对 trustManifest 字段的签名） |

### 7.2 安全考量

| 威胁 | ARD 的缓解措施 |
|------|---------------|
| 伪造 Catalog | 域名验证 + 可选 JWS 签名 |
| 中间人攻击 | HTTPS 传输加密 |
| 恶意 Registry | 任何人可运行 Registry，但 Catalog 来源可独立验证 |
| 隐私泄露 | Catalog 只包含发现元数据，不包含实际数据 |
| 过期资源 | `updatedAt` 字段 + Registry 周期性爬取检查 |

### 7.3 当前实施现状

- **早期 Catalog（Hugging Face、DataRobot）**：仅使用基础的域名锚定
- **中级 Catalog（Suganthan）**：使用 `did:web` 身份标识
- **高级 Catalog（Joost de Valk）**：使用 `did:web` + JWS 签名的 `trustManifest`

---

## 8. 挑战与局限性

### 8.1 技术挑战

| 挑战 | 详细说明 | 严重程度 |
|------|----------|----------|
| **冷启动问题** | 无足够 Catalog → 无搜索价值 → 无人发布 Catalog 的恶性循环 | 🔴 高 |
| **搜索质量** | 纯语义搜索在专业领域（如医疗、法律）的精度尚未验证 | 🟡 中 |
| **实时性** | Catalog 更新 → Registry 重新索引的延迟问题 | 🟡 中 |
| **版本兼容** | 协议 v0.9 但 Schema 版本要求 "1.0" 的错位 | 🟢 低 |
| **跨 Registry 查询** | 分布式 Registry 的联合查询标准和性能未定义 | 🟡 中 |
| **恶意 Catalog 防护** | 无内置机制防止发布恶意/虚假的 Catalog 条目 | 🔴 高 |

### 8.2 生态挑战

| 挑战 | 详细说明 |
|------|----------|
| **OpenAI 和 Anthropic 缺席** | 如果两大关键参与者各自推出竞争标准，将导致生态分裂 |
| **标准成熟度** | v0.9 草案，可能在未来版本中发生 Breaking Change |
| **企业采用意愿** | 企业对"开放标准"的信任需要时间积累 |
| **个人开发者门槛** | 需要在域名根目录部署文件，个人开发者可能被排除在外 |
| **与现有系统的集成** | 企业需改造现有基础设施才能发布 Catalog |

### 8.3 治理挑战

- Linux Foundation 下的工作组治理模式决策速度较慢
- 11 家创始公司利益诉求各不相同，长期方向可能存在分歧
- 缺乏明确的认证/合规机制来保证 Registry 的行为规范

---

## 9. 未来展望与路线图

### 9.1 短期（2026 H2）

| 里程碑 | 预期时间 | 描述 |
|--------|----------|------|
| v1.0 正式版发布 | 2026 Q3-Q4 | 基于 v0.9 反馈迭代 |
| Google Agent Registry ARD 支持 | 未来数月 | Gemini Enterprise Agent Platform 中的原生 ARD 集成 |
| GitHub Copilot Agent Finder 完善 | 进行中 | 扩展对更多 MCP 服务器的发现 |
| 更多创始组织发布 Catalog | 2026 H2 | Google、Microsoft、Salesforce 等预计将发布 |
| hf-discover 持续迭代 | 进行中 | 增加更多 Registry 兼容性 |

### 9.2 中期（2027）

| 方向 | 描述 |
|------|------|
| **大规模公开 Registry** | 出现类似 Web 搜索引擎的大规模 ARD Registry |
| **跨 Registry 联合查询** | 标准化分布式 Registry 查询协议 |
| **企业级信任框架** | SPIFFE/SPIRE 深度集成，零信任架构支持 |
| **开发者工具链完善** | CLI、SDK、CI/CD 集成等工具生态成熟 |
| **开源 Registry 实现** | 除 AGNTCY 外更多可自托管的开源 Registry |

### 9.3 长期（2028+）

| 愿景 | 描述 |
|------|------|
| "Agent 的 Google" | Registry 成为 Agent 的基础设施，就像搜索引擎对 Web 一样 |
| 万物皆可发现 | MCP 服务器、A2A Agent、AI Skills、API、数据集等 |
| 行业垂直 Registry | 医疗、金融、法律等垂直领域的专业 Registry |
| 标准统一 | ARD 成为 Agent 发现的唯一事实标准 |

---

## 10. 落地建议

### 10.1 对于此仓库（ard-catalogs）

| 优先级 | 行动项 | 说明 |
|--------|--------|------|
| 🔴 高 | 持续追踪创始组织的 Catalog 发布状态 | 建立自动化监测，新 Catalog 发布时及时收录 |
| 🔴 高 | 关注 Google、GitHub、Salesforce 的 Catalog 动态 | 这三家的发布将带动生态拐点 |
| 🟡 中 | 增加对 Catalog 的 Schema 验证 | 使用 ajv-cli 和官方 Schema 自动验证收录文件 |
| 🟡 中 | 增加 wasm / container-based 的 Catalog 检查 | 用 hf-discover 定期检查 Catalog 可访问性 |
| 🟢 低 | 将代表性搜索纳入 INDEX.md | 增强索引的可搜索性和实用性 |

### 10.2 对于计划采用 ARD 的团队

| 阶段 | 行动 |
|------|------|
| **第一阶段：学习** | 阅读 ARD 规范，理解 Catalog 和 Registry 的概念 |
| **第二阶段：发布 Catalog** | 在自有域名下部署 `ai-catalog.json`，建议使用 did:web 标识 |
| **第三阶段：集成** | 在 Agent 中集成 ARD 搜索（使用 hf-discover 或其他 Registry） |
| **第四阶段：贡献** | 向此仓库提交 PR，将 Catalog 收录到全球索引中 |

### 10.3 风险提示

1. **标准分裂风险**：如果 OpenAI/Anthropic 推出竞争标准，需评估兼容策略
2. **版本升级风险**：v0.9 → v1.0 可能存在 Breaking Change，建议跟踪规范变更
3. **安全风险**：在不具备 `trustManifest` 的情况下，应对从 Registry 返回的资源保持警惕
4. **依赖风险**：当前生态中 Hugging Face 的 Discover 是唯一可用的注册中心，单点依赖

---

## 11. 参考文献

### 官方规范与文档

- [ARD 官方规范](https://github.com/ards-project/ard-spec) — ARD 协议完整规范
- [ARD JSON Schema](https://github.com/ards-project/ard-spec/blob/main/spec/schemas/ai-catalog.schema.json) — ai-catalog.json 的 JSON Schema 定义
- [ARD 快速开始](https://agenticresourcediscovery.org) — 官方快速入门站点
- [ai-catalog 基础规范](https://github.com/Agent-Card/ai-catalog) — 底层目录格式规范

### 官方公告

- [Google 官方博客：宣布 ARD 规范](https://developers.googleblog.com/en/announcing-the-agentic-resource-discovery-specification/)（2026-06-17）
- [Hugging Face 官方博客：Let agents search](https://huggingface.co/blog/agentic-resource-discovery-launch)（2026-06-17）
- [Cisco Outshift 博客：ARD 如何帮助 Agent 在企业规模下互相发现](https://outshift.cisco.com/blog/ai-ml/agentic-resource-discover-specification-helps-agents-find-each-other)
- [Salesforce 博客：开放发现如何统一企业 Agent](https://www.salesforce.com/blog/open-discovery-agentic-enterprise/)
- [Snowflake 博客：Snowflake 与 ARD 规范](https://www.snowflake.com/en/blog/agentic-resource-discovery-specification/)

### 采用案例

- [DataRobot：Agent Skills 通过 ARD 可发现](https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/)
- [Suganthan：个人 ARD 实现体验](https://suganthan.com/blog/agentic-resource-discovery/)
- [Joost de Valk：ARD 与 OKF 集成介绍](https://joost.blog/okf-ard/)

### 参考实现

- [Hugging Face hf-discover](https://github.com/huggingface/hf-discover) — ARD 客户端/服务器
- [Cisco AGNTCY Agent Directory](https://github.com/agntcy) — ARD 参考实现
- [ARD 教程实验室](https://github.com/Agent-Field/agentic-resource-discovery-lab) — ARD 端到端实操教程

### 媒体报道与分析

- [heise.de：Agentic Resource Discovery — AI Agent 的搜索引擎](https://www.heise.de/en/news/Agentic-Resource-Discovery-Search-engine-for-AI-agents-11338833.html)
- [Search Engine Journal：Google 和 Microsoft 支持 AI Agent 发现草案规范](https://www.searchenginejournal.com/google-microsoft-back-draft-ai-agent-discovery-spec/579894/)
- [ZDNet：AI Agent 拥有了自己的搜索引擎](https://www.zdnet.com/article/ai-agents-are-getting-their-own-search-engine/)
- [WinBuzzer：Google 正在为 AI Agent 构建信任目录](https://winbuzzer.com/2026/06/22/google-ard-release-sharpens-ai-agent-protocol-fight-xcxwbn/)
- [Digital Ocean：A2A vs MCP — 两种 AI Agent 协议的实际区别](https://www.digitalocean.com/community/tutorials/a2a-vs-mcp-ai-agent-protocols)
- [Stream Blog：2026 年顶级 AI Agent 协议 — MCP、A2A、ACP 等](https://getstream.io/blog/ai-agent-protocols/)
- [Stackademic：ARD — AI Agent 缺失的一层](https://blog.stackademic.com/agentic-resource-discovery-ard-the-missing-layer-for-ai-agents-29e2e7542fb7)

### 学术论文

- [From Glue-Code to Protocols: AI Agent Interoperability and the Path to Scalable Multi-Agent Systems (arXiv 2505.03864)](https://arxiv.org/pdf/2505.03864)
- [A Survey of AI Agent Registry Solutions (arXiv 2508.03095)](https://ar5iv.labs.arxiv.org/html/2508.03095)
- [The agent:// Protocol — IETF Draft](https://datatracker.ietf.org/doc/html/draft-narvaneni-agent-uri-03)

---

## 附录

### A. 快速检查清单：是否准备好发布 ARD Catalog？

- [ ] 拥有一个可通过 HTTPS 访问的域名
- [ ] 已搞清楚要发布的资源类型（MCP Server / A2A Agent / AI Skill / API）
- [ ] 已为每个资源分配唯一的 URN（遵循 `urn:air:<publisher>:<namespace>:<name>` 格式）
- [ ] 已生成 `ai-catalog.json` 文件
- [ ] 已使用官方 JSON Schema 验证文件合法性
- [ ] 已部署到 `/.well-known/ai-catalog.json`
- [ ] （可选）已设置 `did:web` 标识
- [ ] （可选）已生成 `trustManifest` 签名
- [ ] 已向 [ard-catalogs](https://github.com/huangxn29/ard-catalogs) 提交收录 PR

### B. 术语对照表

| 英文 | 中文 | 说明 |
|------|------|------|
| ARD (Agentic Resource Discovery) | 代理资源发现协议 | AI Agent 的运行时能力发现标准 |
| Catalog | 目录 | 发布在 `/.well-known/ai-catalog.json` 的静态清单 |
| Registry | 注册中心 | 提供搜索服务的 Catalog 索引 |
| MCP (Model Context Protocol) | 模型上下文协议 | Agent 调用工具的标准化协议 |
| A2A (Agent-to-Agent) | Agent 间通信协议 | Agent 间协作的标准化协议 |
| URN (Uniform Resource Name) | 统一资源名称 | ARD 中资源唯一标识（`urn:air:...`） |
| IANA Media Type | IANA 媒体类型 | 资源类型标识（如 `application/mcp-server-card+json`） |
| trustManifest | 信任清单 | 包含加密签名的安全元数据 |
| did:web | Web DID 方法 | 基于域名可验证标识符 |
| JWS (JSON Web Signature) | JSON Web 签名 | `trustManifest` 的数字签名格式 |
| SPIFFE/SPIRE | 安全身份框架 | 生产环境的加密工作负载标识 |
| AGNTCY | Cisco 的开源 Agent 目录项目 | ARD 的参考实现 |

---

> **文档维护者**: Research Agent (ARD 技术研究员)
> **最后更新**: 2026-06-26
> **下一轮调研**: 建议在 ARD v1.0 正式版发布或主要创始组织（Google、Microsoft）发布 Catalog 时进行更新
