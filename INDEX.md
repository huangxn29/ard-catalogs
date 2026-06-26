# 📋 ARD Catalog 全局索引 / Global Index

> 本索引列出当前仓库收录的所有公开 `ai-catalog.json` 文件。
> Last updated: 2026-06-26

---

## 索引总览 / Overview

| # | 发布者 | 收录路径 | 条目数 | 类型 | 源 URL |
|---|--------|---------|--------|------|--------|
| 1 | Hugging Face | `catalogs/huggingface/ai-catalog.json` | 1 (registry) | `ai-registry` | `https://huggingface.co/.well-known/ai-catalog.json` |
| 2 | DataRobot | `catalogs/datarobot/ai-catalog.json` | 10 (skills) | `ai-skill` | `https://datarobot.com/.well-known/ai-catalog.json` |
| 3 | Suganthan Mohanadasan | `catalogs/suganthan/ai-catalog.json` | 3 (mixed) | `a2a-agent`, `mcp-server`, `openapi` | `https://suganthan.com/.well-known/ai-catalog.json` |
| 4 | Joost de Valk (joost.blog) | `catalogs/joost.blog/ai-catalog.json` | 2 (mixed) | `mcp-server`, `a2a-agent` | `https://joost.blog/.well-known/ai-catalog.json` |

---

## 按类型索引 / Index by Type

### Registry (application/ai-registry+json)

| 发布者 | 标识符 | 端点 URL |
|--------|--------|---------|
| Hugging Face | `urn:ai:huggingface.co:registry:discover` | `https://huggingface-hf-discover.hf.space` |

### Skills (application/ai-skill)

| 发布者 | 标识符 | 描述 |
|--------|--------|------|
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:model-training` | Train models, manage projects, run AutoML |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:model-deployment` | Deploy models, manage deployments |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:predictions` | Make predictions, batch scoring |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:feature-engineering` | Engineer features, feature discovery |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:model-monitoring` | Monitor model performance, data drift |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:model-explainability` | Explain predictions, SHAP values |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:data-preparation` | Upload datasets, manage data assets |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:app-framework-cicd` | CI/CD pipelines for app templates |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:external-agent-monitoring` | OpenTelemetry agent observability |
| DataRobot | `urn:ai:github.com:datarobot-oss:datarobot-agent-skills:agent-assist` | Build and deploy AI agents |

### A2A Agent (application/a2a-agent-card+json)

| 发布者 | 标识符 | 描述 |
|--------|--------|------|
| Suganthan | `urn:ai:suganthan.com:agent:blog-agent` | Blog search and retrieval agent |
| Joost de Valk | `urn:ai:joost.blog:a2a:ask-joost` | Ask questions about SEO, WordPress, AI, and the open web via Joost de Valk's content |

### MCP Server (application/mcp-server-card+json)

| 发布者 | 标识符 | 描述 |
|--------|--------|------|
| Suganthan | `urn:ai:suganthan.com:server:blog-mcp` | Blog MCP server with search tools |
| Joost de Valk | `urn:ai:joost.blog:mcp:joost-blog` | MCP server for searching Joost de Valk's blog content |

### OpenAPI (application/vnd.oai.openapi+json)

| 发布者 | 标识符 | 描述 |
|--------|--------|------|
| Suganthan | `urn:ai:suganthan.com:api:blog-openapi` | Blog REST API |

---

## 按发布者详情 / Publisher Details

### Hugging Face

- **域名**: `huggingface.co`
- **主页**: https://huggingface.co
- **Catalog URL**: https://huggingface.co/.well-known/ai-catalog.json
- **收录文件**: `catalogs/huggingface/ai-catalog.json`
- **收录日期**: 2026-06-26
- **验证状态**: ✅ 可通过 HTTP 访问
- **备注**: 指向动态搜索注册中心，实际资源需通过 Discover Registry 查询
- **相关工具**: https://github.com/huggingface/hf-discover

### DataRobot

- **域名**: `datarobot.com`
- **主页**: https://www.datarobot.com
- **Catalog URL**: https://datarobot.com/.well-known/ai-catalog.json
- **收录文件**: `catalogs/datarobot/ai-catalog.json`
- **收录日期**: 2026-06-26
- **验证状态**: ✅ 可通过 HTTP 访问
- **备注**: 10 个 Agent Skills，覆盖 MLOps 全生命周期
- **相关博客**: https://www.datarobot.com/blog/datarobot-agent-skills-are-now-discoverable-through-agentic-resource-discovery/

### Suganthan Mohanadasan

- **域名**: `suganthan.com`
- **主页**: https://suganthan.com
- **Catalog URL**: https://suganthan.com/.well-known/ai-catalog.json
- **收录文件**: `catalogs/suganthan/ai-catalog.json`
- **收录日期**: 2026-06-26
- **验证状态**: ✅ 可通过 HTTP 访问
- **备注**: 完整使用 `representativeQueries`、`capabilities`、`version` 等高级特性
- **相关博客**: https://suganthan.com/blog/agentic-resource-discovery/

### Joost de Valk (joost.blog)

- **域名**: `joost.blog`
- **主页**: https://joost.blog
- **Catalog URL**: https://joost.blog/.well-known/ai-catalog.json
- **收录文件**: `catalogs/joost.blog/ai-catalog.json`
- **收录日期**: 2026-06-26
- **验证状态**: ✅ 可通过 HTTP 访问
- **备注**: Joost de Valk (Yoast 创始人) 发布了一个 Ard catalog，包含一个 MCP server 和一个 A2A agent，覆盖 SEO、WordPress、AI 和开放网络等主题。使用 did:web identity 和 JWS 签名的 trustManifest。
- **相关博客**: https://joost.blog/okf-ard/

---

## 示例文件 / Examples

| 文件 | 描述 |
|------|------|
| `examples/enterprise-catalog.json` | 来自 ARD 规范的完整企业级 Catalog 示例 |
| `examples/solo-developer-catalog.json` | 来自 ARD 规范的个人开发者 Catalog 示例 |

---

## 语言版本 / Language Versions

| 语言 | 文件 |
|------|------|
| 🇨🇳 中文 | [`README.zh.md`](README.zh.md) |
| 🇬🇧 English | [`README.md`](README.md) |

---

## 关注我们 / Follow Us

在 X/Twitter 上关注 [@huangxn29](https://x.com/huangxn29) 获取最新动态。
Follow [@huangxn29](https://x.com/huangxn29) on X/Twitter for the latest updates.

---

## 统计 / Statistics

| 指标 | 数值 |
|------|------|
| 已收录发布者 | 4 |
| 总条目数 | 16 |
| Registry 条目 | 1 |
| Skill 条目 | 10 |
| A2A Agent 条目 | 2 |
| MCP Server 条目 | 2 |
| OpenAPI 条目 | 1 |
| 已验证可访问 | 4/4 (100%) |
