---
title: "解析引擎速查"
---

## 概述

解析引擎负责将上传的文档（PDF、Word、Excel 等）转换为可检索的文本内容。不同文档类型适合不同的解析引擎，选择合适的引擎可以显著提升文档入库速度和解析质量。本文汇总各文档类型推荐引擎、引擎对比和配置建议。

---

## 前置条件

- 已登录睿阁并拥有管理员权限（配置解析引擎需要管理员角色）。
- 已在[全局设置 → 解析引擎](/zh/reference/04-settings)中了解解析引擎管理入口。

---

## 文档类型配置速查表

| 文档类型 | 推荐解析引擎 | 其他可用引擎 |
|---------|------------|------------|
| PDF 文档 | MinerU Cloud（推荐） | 自部署 MinerU / PaddleOCR-VL Cloud / PaddleOCR-VL / DocReader（内置） |
| Word 文档 | MinerU Cloud（推荐） | 自部署 MinerU / DocReader（内置） |
| PPT | MinerU Cloud（推荐） | 自部署 MinerU / DocReader（内置） |
| Excel 表格 | MarkItDown（推荐） | DocReader（内置） |
| CSV | Simple（内置） | MarkItDown |
| Markdown | DocReader（内置） | Simple（内置）/ MarkItDown |
| TXT | Simple（内置） | — |
| JSON | Simple（内置） | — |
| 图片 | MinerU Cloud（推荐） | 自部署 MinerU / PaddleOCR-VL Cloud / PaddleOCR-VL / Simple（仅提取 EXIF） |
| 音频 | Simple（内置） | — |

---

## 解析引擎对比

| 引擎 | 类型 | 说明 | 适用场景 |
|------|------|------|---------|
| DocReader（内置/builtin） | 内置 | 支持 docx/pdf/xlsx 等复杂格式，通过 gRPC 或 HTTP 调用 | 通用文档解析，开箱即用 |
| Simple | 内置 | 简单文本提取，速度快 | TXT、JSON、CSV 等纯文本文件 |
| MarkItDown | 内置 | Microsoft 文档转换工具，支持 PDF/Office/HTML | Excel 表格、HTML 等格式 |
| MinerU Cloud | 云端 | MinerU 云服务，开箱即用，免费额度每日 1,000 页 | PDF、Word、PPT、图片等复杂文档 |
| 自部署 MinerU | 自建 | 本地 GPU 加速解析，支持 5 种后端模式 | 大量文档解析、追求速度 |
| PaddleOCR-VL Cloud | 云端 | 百度飞桨云服务，每日免费 10,000 页 | 财务报表、合同、扫描件 |
| PaddleOCR-VL | 自建 | 本地部署 PaddleOCR | 需要本地化的场景 |
| OpenDataLoader | 内置 | 版面分析引擎，需 Java 11+ | PDF 版面分析 |

---

## 复杂 PDF 选择建议

| 文档类型 | 推荐引擎 | 配置建议 |
|---------|---------|---------|
| 学术论文/技术文档 | MinerU | 启用公式识别、表格识别，使用 pipeline 或 hybrid-auto-engine |
| 财务报表/合同 | PaddleOCR-VL Cloud | 启用印章识别、图表识别 |
| 扫描件/图片 PDF | MinerU 或 PaddleOCR-VL Cloud | 启用 OCR，设置正确的语言 |
| 混合内容文档 | MinerU | 使用 hybrid-auto-engine |

---

## MinerU Cloud vs 自部署对比

| 对比项 | MinerU Cloud | 自部署 MinerU |
|--------|-------------|--------------|
| 部署难度 | 无需部署，开箱即用 | 需要 GPU 服务器和配置 |
| 免费额度 | 每日 1,000 页（30,000 页/月） | 无限制 |
| 文件大小限制 | 最大 10MB | 无限制 |
| 页数限制 | 最大 200 页 | 无限制 |
| 解析速度（200 页 PDF） | 约 2 小时 | 17 分钟（A10）/ 5-8 分钟（2×A100） |
| 并发能力 | 无限（云服务排队） | 2-8 并发（取决于 GPU） |
| 适用场景 | 小规模、试用阶段 | 大量文档、追求速度 |

> 注意：MinerU Cloud 单文件不超过 10MB、200 页。超出限制的文件请切换为自部署 MinerU 或 PaddleOCR-VL Cloud。

---

## 进阶用法

### MinerU Cloud Model Version

MinerU Cloud 支持 3 种模型版本：

| Model Version | 说明 | 适用场景 |
|---------------|------|---------|
| pipeline | 经典管道模式，多模型级联 | 通用文档、显存有限 |
| vlm | 视觉语言模型，单次推理 | 复杂文档、追求精度 |
| MinerU-HTML | HTML 输出格式 | 需要保留 HTML 结构 |

### 自部署 MinerU Backend 选项

自部署支持 5 种后端模式，详见 [预算与配置指南](/zh/reference/02-budget-guide)。

---

## 常见问题

### Q1：图片解析后只显示文件，没有摘要内容怎么办？

MinerU 负责提取文档文本和识别图片，但图片描述需要多模态大模型（VLM）生成。请检查是否配置了可用的 VLM 模型，或尝试切换解析引擎。

### Q2：MinerU Cloud 解析失败怎么办？

常见原因：文档超过 200 页或文件超过 10MB。解决方案：
- 切换为自部署 MinerU。
- 切换为 PaddleOCR-VL Cloud。
- 拆分文件后重新上传。

### Q3：.doc 文档解析失败怎么办？

尝试将 .doc 文件另存为一份新的 .doc 文档，再重新上传解析。重新保存可解决部分格式兼容问题。

### Q4：CSV 文件解析后显示乱码怎么办？

保存 CSV 文件时使用 UTF-8 + BOM 编码方式，然后重新上传。

### Q5：自部署 MinerU 的并发限制是多少？

默认同时处理 3 个解析任务，第 4 个会失败。单卡建议配置 2 个并发。可通过环境变量 `MINERU_API_MAX_CONCURRENT_REQUESTS` 调整。

---

## 相关文档

- 前置阅读：
  - [模型配置速查](/zh/reference/01-model-config)（VLM 模型选择）
- 关联操作：
  - [预算与配置指南](/zh/reference/02-budget-guide)（MinerU 自部署配置和硬件方案）
  - [全局设置](/zh/reference/04-settings)（解析引擎配置入口）
  - [常见问题](/zh/faq)（更多解析相关问题）

---

> 最后更新：2026-09-09 | 对应版本：v1.0.0
