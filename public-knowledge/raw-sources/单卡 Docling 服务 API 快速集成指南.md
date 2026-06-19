---
id: 7F30B049-7505-400F-B5F3-B1F95E9CBCFA
title: 单卡 Docling 服务 API 快速集成指南
status: draft
created-at: 2026-06-19T00:37:25Z
updated-at: 2026-06-19T00:37:47Z
---

# 单卡 Docling 服务 API 快速集成指南

当前单卡 Docling Serve 适合单文件同步解析、功能验证和需要使用 chunk/async/source 等原生能力的场景。

```text
Docling Serve URL：http://10.102.76.35:18081
Docling Serve Version: docling-serve 1.21.0, docling 2.96.1
```

Docling Serve OpenAPI 暴露的输入格式：

```text
docx, pptx, html, image, pdf, asciidoc, md, csv, xlsx,
xml_uspto, xml_jats, xml_xbrl, mets_gbs, json_docling,
audio, vtt, latex
```

输出格式：

```text
md, json, yaml, html, html_split_page, text, doctags, vtt
```

常用参数：

```text
to_formats            输出格式，例如 ["md"] 或 ["md","json"]
from_formats          限定输入格式，通常可省略
do_ocr                是否启用 OCR，扫描件和图片建议 true
force_ocr             是否强制用 OCR 覆盖已有文本，默认 false
ocr_lang              OCR 语言列表
ocr_preset            OCR 预设，例如 auto/easyocr/tesseract
do_table_structure    是否识别表格结构
table_mode            fast 或 accurate
table_cell_matching   是否将预测表格单元格匹配回 PDF 单元格
pdf_backend           pypdfium2/docling_parse/threaded_docling_parse/dlparse_v1/dlparse_v2/dlparse_v4
pipeline              standard/legacy/vlm/asr
page_range            页码范围，页码从 1 开始，例如 [1,10]
document_timeout      单文档处理超时秒数
image_export_mode     placeholder/embedded/referenced
target_type           inbody 或 zip
```

说明：

* `to_formats` 默认是 `["md"]`，所以只传 `target_type=zip` 不会自动生成 JSON 文件。
* multipart 请求里多个输出格式应重复传 `to_formats` 字段，例如 `-F 'to_formats=md' -F 'to_formats=json'`。
* 独立图片文件依赖 `image_export_mode=referenced` 和文档中实际可导出的图片/图形。
* Docling referenced 图片目录名为 `artifacts/`；这和 MinerU 常见的 `images/` 目录命名不同。

## 健康和版本

```bash
curl -fsS http://<host>:18081/health
curl -fsS http://<host>:18081/ready
curl -fsS http://<host>:18081/version
curl -fsS http://<host>:18081/docs
```

## 同步文件解析

```bash
curl -fsS -X POST http://<host>:18081/v1/convert/file \
  -F 'files=@sample.pdf;type=application/pdf' \
  -F 'to_formats=md' \
  -F 'do_ocr=true' \
  -F 'do_table_structure=true' \
  -F 'table_mode=accurate' \
  -F 'target_type=inbody'
```

`files` 支持多文件。

如果希望直接拿到 zip 文件，将 `target_type` 设为 `zip`，并把二进制响应保存到本地文件：

```bash
curl -fS -X POST http://<host>:18081/v1/convert/file \
  -F 'files=@sample.pdf;type=application/pdf' \
  -F 'to_formats=md' \
  -F 'do_ocr=true' \
  -F 'do_table_structure=true' \
  -F 'target_type=zip' \
  -o converted_docs.zip

unzip -l converted_docs.zip
```

如果需要同时得到 Markdown 和 Docling JSON，必须显式传多个 `to_formats` 表单字段：

```bash
curl -fS -X POST http://<host>:18081/v1/convert/file \
  -F 'files=@sample.pdf;type=application/pdf' \
  -F 'to_formats=md' \
  -F 'to_formats=json' \
  -F 'do_ocr=true' \
  -F 'do_table_structure=true' \
  -F 'target_type=zip' \
  -o converted_docs.zip
```

如果需要单独图片文件，使用 referenced 图片模式：

```bash
curl -fS -X POST http://<host>:18081/v1/convert/file \
  -F 'files=@sample.pdf;type=application/pdf' \
  -F 'to_formats=md' \
  -F 'to_formats=json' \
  -F 'include_images=true' \
  -F 'image_export_mode=referenced' \
  -F 'target_type=zip' \
  -o converted_docs_with_artifacts.zip
```

`image_export_mode=referenced` 生成的图片目录名是 `artifacts/`，不是 MinerU 常见的 `images/`：

如果使用默认 `image_export_mode=embedded`，图片通常嵌入 Markdown/JSON 内容或以内部引用表达，不会额外生成独立图片目录。Docling 的输出目录结构与 MinerU 不完全一致，不能假定一定存在 `images/` 文件夹。

## 同步 URL 解析

```bash
curl -fsS -X POST http://<host>:18081/v1/convert/source \
  -H 'Content-Type: application/json' \
  -d '{
    "sources": [
      {"kind": "http", "url": "https://example.com/sample.pdf"}
    ],
    "options": {
      "to_formats": ["md"],
      "do_ocr": true,
      "do_table_structure": true
    }
  }'
```

外部 URL 会经过 Docling Serve 的 URL 安全校验。若业务系统能直接上传文件，优先使用文件上传接口，避免外部 URL、代理、DNS 和内网地址限制导致解析失败。

## 异步接口

提交：

```bash
curl -fsS -X POST http://<host>:18081/v1/convert/file/async \
  -F 'files=@sample.pdf;type=application/pdf' \
  -F 'to_formats=md' \
  -F 'do_ocr=true' \
  -F 'do_table_structure=true'
```

轮询：

```bash
curl -fsS 'http://<host>:18081/v1/status/poll/<task_id>?wait=0'
```

读取：

```bash
curl -fsS http://<host>:18081/v1/result/<task_id>
```

注意：Docling Serve 原生异步结果是服务内结果管理。

## Chunk 接口

Docling Serve 还提供 hybrid/hierarchical chunk 接口，常用于 RAG 入库前切块。

```text
POST /v1/chunk/hybrid/file
POST /v1/chunk/hybrid/file/async
POST /v1/chunk/hybrid/source
POST /v1/chunk/hybrid/source/async
POST /v1/chunk/hierarchical/file
POST /v1/chunk/hierarchical/file/async
POST /v1/chunk/hierarchical/source
POST /v1/chunk/hierarchical/source/async
```

示例：

```bash
curl -fsS -X POST http://<host>:18081/v1/chunk/hybrid/file \
  -F 'files=@sample.pdf;type=application/pdf' \
  -F 'convert_do_ocr=true' \
  -F 'convert_do_table_structure=true' \
  -F 'chunking_max_tokens=512' \
  -F 'include_converted_doc=false'
```

Chunk 接口返回切块结果；如需同时返回转换后的文档，可设置 `include_converted_doc=true`。`file/source` 和 `sync/async` 只是输入来源与调用方式的差异，不是额外切片算法。

结合当前容器内源码，Docling Serve 的 chunk 策略分为两类：

| 策略                    | 公开接口                       | 核心行为                                                           | 典型用途                     |
| :-------------------- | :------------------------- | :------------------------------------------------------------- | :----------------------- |
| `HybridChunker`       | `/v1/chunk/hybrid/*`       | 先按文档结构生成基础块，再用 tokenizer 做 token-aware 拆分，并可合并同标题下的相邻小块        | RAG 入库，尤其需要控制每块 token 数时 |
| `HierarchicalChunker` | `/v1/chunk/hierarchical/*` | 主要按 DoclingDocument 的标题、章节、列表、正文项、表格等结构输出块，不按 tokenizer 强制限制长度 | 希望最大程度保留文档层级和原始结构边界时     |

常用 chunk 参数：

```text
两类策略共有:
  chunking_use_markdown_tables    表格是否输出为 markdown table；默认 false，默认更偏 triplet 文本
  chunking_use_markdown_images    是否在 chunk 中序列化图片引用；默认 false
  chunking_image_placeholder      图片占位符；默认 ![IMAGE]
  chunking_include_raw_text       是否同时返回 raw_text；默认 false
  include_converted_doc           是否在 chunk 结果中包含转换后的文档；默认 false

仅 HybridChunker:
  chunking_max_tokens             每块最大 token 数；为空时从 tokenizer 读取
  chunking_tokenizer              HuggingFace tokenizer 名称；默认 sentence-transformers/all-MiniLM-L6-v2
  chunking_merge_peers            是否合并同 headings 的相邻小块；默认 true
```

`HybridChunker` 会返回 `num_tokens`；`HierarchicalChunker` 不做 tokenizer 计数，`num_tokens` 通常为空。`include_converted_doc=true` 时，inbody 响应会带转换后的文档内容；zip 目标会生成 `chunked_result.json`，若图片使用 referenced 模式，还会在临时输出目录生成 `artifacts/` 后打包。`InBodyTarget` 不支持 `image_export_mode=referenced`。
