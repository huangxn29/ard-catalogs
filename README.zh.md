<!--
  ard-catalogs — ARD Catalog 宝典
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  Agentic Resource Discovery (ARD) 协议的公开 ai-catalog.json 集合仓库。
  中文版本。
-->

<p align="center">
  <img src="https://agenticresourcediscovery.org/favicon.ico" width="64" />
</p>

# 🗂️ ARD Catalog 宝典

> 收集公开高质量 `ai-catalog.json`，使本仓库成为"ARD Catalog 宝典"

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![ARD Spec v0.9](https://img.shields.io/badge/ARD-v0.9%20(Draft)-purple)](https://github.com/ards-project/ard-spec)
[![Star History](https://img.shields.io/badge/Star-History-gold?logo=github)](https://star-history.com/#huangxn29/ard-catalogs&Date)
[![关注 @huangxn29](https://img.shields.io/badge/X-@huangxn29-000000?logo=x)](https://x.com/huangxn29)

[English version →](README.md)

> **在 X/Twitter 上关注 [@huangxn29](https://x.com/huangxn29)**，获取最新的 Catalog 收录动态、ARD 生态进展等更多信息。

---

## 📖 介绍

**Agentic Resource Discovery (ARD)** 是一个开放、联合的去中心化发现协议，由 **Google、Microsoft、Hugging Face、GitHub、NVIDIA、Snowflake、Salesforce、Cisco、Databricks、GoDaddy、ServiceNow** 等公司共同支持（Linux Foundation 托管），于 2026 年 6 月 17 日发布草案 v0.9。

ARD 定义了一种标准方式，让 AI Agent 能够通过发布在 `/.well-known/ai-catalog.json` 的机器可读清单，动态发现工具、MCP 服务器、A2A Agent 和 API。

本仓库旨在：
- ✅ **收集** 互联网上已公开的 `ai-catalog.json` 文件
- ✅ **维护** 这些文件的精确副本和元数据
- ✅ **索引** 提供全局索引便于查阅
- ✅ **教学** 包含规范的 JSON Schema 和示例文件
- ✅ **追踪** 记录各个发布者的采用进展

---

## 📂 仓库结构

```
ard-catalogs/
├── catalogs/               # 收集的 ai-catalog.json 文件（按发布者组织）
│   ├── huggingface/        # Hugging Face — huggingface.co
│   ├── datarobot/          # DataRobot — datarobot.com
│   ├── suganthan/          # Suganthan (个人开发者) — suganthan.com
│   └── joost.blog/         # Joost de Valk (Yoast 创始人) — joost.blog
├── docs/                   # 调研报告与文档
│   └── ard-deep-research-report.md  # ARD 协议深度调研报告
├── spec/                   # ARD 协议规范文档
│   └── schemas/            # JSON Schema 定义
├── examples/               # 示例 catalog 文件
├── INDEX.md                # 全局索引
└── README.md               # 本文件
```

---

## 🌐 已收录的 Catalog

| # | 发布者 | 域名 | Catalog URL | 资源类型 | 收录日期 |
|---|--------|------|-------------|----------|---------|
| 1 | **Hugging Face** | huggingface.co | [🔗](https://huggingface.co/.well-known/ai-catalog.json) | ai-registry | 2026-06-29 |
| 2 | **DataRobot** | datarobot.com | [🔗](https://datarobot.com/.well-known/ai-catalog.json) | ai-skill | 2026-06-29 |
| 3 | **Suganthan Mohanadasan** | suganthan.com | [🔗](https://suganthan.com/.well-known/ai-catalog.json) | a2a-agent, mcp-server, openapi | 2026-06-29 |
| 4 | **Joost de Valk** | joost.blog | [🔗](https://joost.blog/.well-known/ai-catalog.json) | mcp-server, a2a-agent | 2026-06-29 |

> **注意**: ARD 协议于 2026 年 6 月 17 日刚发布 v0.9 草案，目前只有少数先锋组织和开发者发布了公开的 ai-catalog.json。随着生态成熟，本仓库将持续更新收录。

---

## 📋 Catalog 详解

### 1. Hugging Face

- **URL**: `https://huggingface.co/.well-known/ai-catalog.json`
- **类型**: Registry（注册中心）
- **描述**: Hugging Face 发布了指向其 Discover 注册中心的 ARD 入口，索引了 Hugging Face 上的 Skills、Space Skills 和 MCP 启用的 Spaces
- **标识符**: `urn:ai:huggingface.co:registry:discover`
- **注册中心端点**: `https://huggingface-hf-discover.hf.space`
- **CLI 工具**: `hf-discover`（开源，[GitHub](https://github.com/huggingface/hf-discover)）

### 2. DataRobot

- **URL**: `https://datarobot.com/.well-known/ai-catalog.json`
- **类型**: Skills（技能集合）
- **描述**: DataRobot 发布了 10 个 Agent Skills，覆盖模型训练、部署、预测、特征工程、模型监控、可解释性、数据准备、CI/CD、外部 Agent 监控和 Agent 构建
- **所有条目**均为 `application/ai-skill` 类型
- **参考**: [DataRobot 官方博客](https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/)

### 3. Suganthan Mohanadasan

- **URL**: `https://suganthan.com/.well-known/ai-catalog.json`
- **类型**: 混合（A2A Agent + MCP Server + OpenAPI）
- **描述**: 个人开发者部署的完整 ARD 实现，包含 3 个条目：
  - A2A Blog Agent（博客搜索 Agent）
  - Blog MCP Server（MCP 服务）
  - Blog REST API（OpenAPI 接口）
- 使用了 `representativeQueries`、`capabilities`、`version` 等高级字段
- 使用了 `did:web` 作为 host 标识符
- **参考**: [Suganthan 博客](https://suganthan.com/blog/agentic-resource-discovery/)

### 4. Joost de Valk (joost.blog)

- **URL**: `https://joost.blog/.well-known/ai-catalog.json`
- **类型**: 混合（MCP Server + A2A Agent）
- **描述**: Joost de Valk — Yoast 的创始人 — 发布了一个包含 2 个条目的 ARD catalog：
  - MCP Server：搜索关于 SEO、WordPress、AI 和开放网络的博客内容
  - A2A Agent：基于他公开发布的文章和演讲内容回答自然语言问题
- 使用了 `did:web` 身份标识和包含 JWS 签名的 `trustManifest`
- **参考**: [Joost 博客](https://joost.blog/okf-ard/)

---

## 📐 ARD 协议概述

| 属性 | 值 |
|------|-----|
| **全称** | Agentic Resource Discovery (ARD) |
| **版本** | v0.9 (Draft) |
| **发布时间** | 2026-06-17 |
| **许可** | Apache 2.0 |
| **托管** | Linux Foundation (AI Catalog Working Group) |
| **核心文件** | `/.well-known/ai-catalog.json` |
| **URN 格式** | `urn:air:<publisher>:<namespace>:<agent-name>` |
| **支持者** | Google, Microsoft, Hugging Face, GitHub, NVIDIA, Snowflake, Salesforce, Cisco, Databricks, GoDaddy, ServiceNow |

### 核心设计原则

1. **Search-First Discovery**: Agent 通过搜索动态发现，而非预安装
2. **Scalability Beyond Context Windows**: 将选择问题从 LLM context window 移至专用搜索服务
3. **Artifact Agnostic Envelope**: 使用 IANA Media Type 标识资源类型，不约束内部 schema
4. **Value-or-Reference**: 条目必须包含 `url`（远程引用）或 `data`（内嵌）之一
5. **REST API 基准**: 注册中心必须暴露 HTTP REST 搜索接口

### 资源类型（IANA Media Types）

| 类型 | 含义 |
|------|------|
| `application/mcp-server-card+json` | MCP 服务器 |
| `application/a2a-agent-card+json` | A2A Agent |
| `application/ai-skill` | AI Skill |
| `application/ai-catalog+json` | 嵌套子 Catalog |
| `application/ai-registry+json` | 支持搜索的动态注册中心 |
| `application/vnd.oai.openapi+json` | OpenAPI 描述 |

---

## 🔍 扫描状态

以下列出了 ARD 创始公司/组织的 `/.well-known/ai-catalog.json` 可访问性扫描结果（2026-06-29）：

| 组织 | 域名 | 状态 | 备注 |
|------|------|------|------|
| ✅ Hugging Face | huggingface.co | **已发布** | 指向 Discover 注册中心 |
| ✅ DataRobot | datarobot.com | **已发布** | 10 个 Agent Skills |
| ✅ Suganthan | suganthan.com | **已发布** | 个人开发者，3 个条目 |
| ✅ Joost de Valk | joost.blog | **已发布** | 个人开发者，2 个条目（MCP + A2A） |
| ⬜ Google | google.com | 404 | 未发布 |
| ⬜ Google Cloud | cloud.google.com | 404 | 未发布 |
| ⬜ Microsoft | microsoft.com | 404 | 未发布 |
| ⬜ GitHub | github.com | 404 | 未发布（Agent Finder 已进入 Copilot） |
| ⬜ Snowflake | snowflake.com | 404 | 未发布 |
| ⬜ Salesforce | salesforce.com | 无响应 | 未发布 |
| ⬜ Databricks | databricks.com | 404 | 未发布 |
| ⬜ NVIDIA | nvidia.com | 403 | 未公开 |
| ⬜ GoDaddy | godaddy.com | 403 | 未公开 |
| ⬜ ServiceNow | servicenow.com | 403 | 未公开 |
| ⬜ Cisco | cisco.com | 404 | 未发布 |
| ⬜ Anthropic | anthropic.com | 无响应 | 非创始成员 |
| ⬜ OpenAI | openai.com | 无响应 | 非创始成员 |

> 截止目前，**仅 Hugging Face、DataRobot、Suganthan 和 Joost de Valk** 已发布公开的 `ai-catalog.json`。ARD 生态尚处于早期阶段。

---

## 🚀 快速开始

### 验证 Catalog 合法性

使用 ARD 官方 JSON Schema 验证：

```bash
npx ajv-cli validate -s spec/schemas/ai-catalog.schema.json -d catalogs/huggingface/ai-catalog.json
```

### 使用 hf-discover 搜索

```bash
# 安装 hf-discover
pip install hf-discover

# 搜索 Hugging Face 生态中的 MCP 服务器
hf discover search "transcribe some audio" --kind mcp

# 导航到任意网站并发现其 ARD catalog
hf discover navigate suganthan.com "find an MCP server for blog search"
```

---

## 🤝 贡献指南

欢迎贡献！详细指南请见 [CONTRIBUTING.md](CONTRIBUTING.md)。

如果你发现了新的公开 `ai-catalog.json`，请：
1. Fork 本仓库
2. 在 `catalogs/<publisher-name>/` 目录下添加新的 catalog 文件
3. 更新 `INDEX.md` 中的索引
4. 提交 Pull Request

### 准则

- 只收录**公开可访问**的 `ai-catalog.json`（无需认证）
- 保持 JSON 文件的**原始格式**，不修改内容
- 在目录中添加 `meta.json` 记录来源 URL、抓取日期和备注
- 如果某个 catalog 已经下线或变更，请更新状态

---

## 📚 参考资料

- [ARD 官方规范](https://github.com/ards-project/ard-spec) — 完整的 ARD 协议规范
- [ARD JSON Schema](spec/schemas/ai-catalog.schema.json) — ai-catalog.json 的 JSON Schema 定义
- [ai-catalog 基础规范](https://github.com/Agent-Card/ai-catalog) — 底层目录格式规范
- [Hugging Face Discover](https://github.com/huggingface/hf-discover) — Hugging Face 的 ARD 客户端/服务器
- [ARD 教程实验室](https://github.com/Agent-Field/agentic-resource-discovery-lab) — ARD 协议实操教程
- [Hugging Face 官方博客](https://huggingface.co/blog/agentic-resource-discovery-launch) — ARD 发布博文
- [DataRobot 官方博客](https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/) — DataRobot ARD 集成博文
- [Suganthan 博客](https://suganthan.com/blog/agentic-resource-discovery/) — 个人开发者 ARD 实现体验
- [Joost 博客](https://joost.blog/okf-ard/) — Joost de Valk 的 ARD 与 OKF 集成介绍

---

## ⭐ 支持

如果你觉得这个项目有用，请在 GitHub 上点一个 Star —— 这能帮助更多人发现 ARD 生态！

[![Star History Chart](https://api.star-history.com/svg?repos=huangxn29/ard-catalogs&type=Date)](https://star-history.com/#huangxn29/ard-catalogs&Date)

---

## 📄 许可

本仓库本身使用 Apache 2.0 许可。收录的各 `ai-catalog.json` 文件版权归其原始发布者所有，仅在教育/参考目的下收录。
