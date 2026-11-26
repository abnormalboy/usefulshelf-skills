---
name: usefulshelf-tools
description: UsefulShelf (usefulshelf.com / 实用工具架) agent toolkit — stateless HTTP MCP gateway (/api/mcp) for 5 deterministic developer tools, HTTP 402 batch endpoint (/api/mcp/batch, up to 50 items), machine discovery (/llms.txt, /llms-full.txt, /api/skills/manifest), CLI skill install (npx skills add abnormalboy/usefulshelf-skills), 12 in-browser W3C WebMCP tools (window.navigator.modelContext), and 60 bilingual zero-upload web utilities.
---

# UsefulShelf Agent & Developer Toolkit (`usefulshelf-tools`)

> **中文速览**：UsefulShelf（实用工具架，`https://usefulshelf.com`）提供免密钥无状态 HTTP MCP 网关（`/api/mcp`，5 个确定性开发工具）、支持单次最多 50 项批处理的 HTTP 402 批量接口（`/api/mcp/batch`）、机器可读目录（`/llms.txt` 与 `/llms-full.txt`）、12 个浏览器端 W3C WebMCP 工具以及 60 款中英双语、本地优先零上传的在线实用工具。

Use this skill whenever an AI coding agent or developer workflow needs **100% deterministic, zero-hallucination data transformations** (JSON repair/validation, 5-field Cron scheduling, JWT claim inspection, UTM normalization, cryptographic hashing/HMAC), **browser-local zero-upload file/document workbenches**, or **installable interactive UI skills**.

---

## 1. Machine Discovery & Quick Install

| Resource | Endpoint / Command | Notes |
| --- | --- | --- |
| Stateless MCP Gateway | `https://usefulshelf.com/api/mcp` | Streamable HTTP JSON-RPC 2.0 (`2025-06-18`, `2025-03-26`, `2024-11-05`), no API key, zero persistence |
| Batch MCP Endpoint | `https://usefulshelf.com/api/mcp/batch` | Up to 50 items per request, HTTP 402 credit flow |
| Concise Machine Index | `https://usefulshelf.com/llms.txt` | Plain-text overview of endpoints, 12 WebMCP tools, and all 60 web tools |
| Full Schema Directory | `https://usefulshelf.com/llms-full.txt` | Full JSON Schemas for HTTP MCP, HTTP 402 batch, WebMCP, and all Agent Skills |
| Skills JSON Manifest | `https://usefulshelf.com/api/skills/manifest` | Structured catalog of SKILL.md and deterministic ZIP archive URLs |
| Interactive Web Suite | `https://usefulshelf.com/tools` (`/en/tools`) | 60 live bilingual browser tools (local-first zero upload) |

### Install Agent Skills via CLI
```bash
npx skills add abnormalboy/usefulshelf-skills
```

### Add Stateless MCP Server in Claude Code
```bash
claude mcp add --transport http usefulshelf https://usefulshelf.com/api/mcp
```

### Cursor / Windsurf / Generic `mcp.json`
```json
{
  "mcpServers": {
    "usefulshelf": {
      "type": "http",
      "url": "https://usefulshelf.com/api/mcp"
    }
  }
}
```

### Download This `SKILL.md` Locally
```bash
mkdir -p .agents/skills/usefulshelf-tools
curl -fsSL https://usefulshelf.com/skills/usefulshelf-tools/SKILL.md \
  -o .agents/skills/usefulshelf-tools/SKILL.md
```

---

## 2. Stateless MCP Gateway (`/api/mcp`)

- **GET `https://usefulshelf.com/api/mcp`**: Returns service metadata, supported protocol versions, tool summaries, WebMCP catalog, and quick-start snippets.
- **POST `https://usefulshelf.com/api/mcp`**: Streamable HTTP JSON-RPC 2.0 (`Content-Type: application/json`, max body `256 KiB`, max `1 MiB` per text field, supports JSON-RPC array batches of 1–25 requests).
- **Methods**: `initialize`, `ping`, `tools/list`, `tools/call`.
- **Annotations on all 5 tools**: `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: false`.

### Tool Schemas (Exact `additionalProperties: false` Contracts)

| Tool Name | Purpose | Required & Optional Arguments |
| --- | --- | --- |
| `json_format_validate` | Validate JSON with exact `line`/`column`/`position`, format or minify, sort object keys, or safely repair loose JSON (comments, single quotes, trailing commas) | `input` (string, **required**), `indent` (`2 \| 4`, default `2`), `sortKeys` (boolean, default `false`), `compact` (boolean, default `false`), `repair` (boolean, default `false`) |
| `cron_explain` | Validate a 5-field cron expression (`min hour dom mon dow`), output ZH + EN field explanations, and compute upcoming ISO-8601 run timestamps | `expression` (string, **required**), `count` (integer `1..20`, default `5`), `afterIso` (optional ISO-8601 string) |
| `jwt_decode_inspect` | Decode JWT header & payload claims (auto-strips `Bearer `), convert `iat`/`nbf`/`exp` to ISO-8601, and flag expired or unsigned (`alg=none`) tokens | `token` (string, **required**), `nowEpochSeconds` (optional number) |
| `utm_link_build` | Normalize HTTP/HTTPS URLs and attach `utm_*` + extra query parameters while preserving existing query keys and URL fragments | `url` (string, **required**), `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content` (strings), `extraParams` (`Record<string, string>`) |
| `hash_digest` | Compute MD5, SHA-1, SHA-256, and SHA-512 checksums or HMAC digests in lowercase hex, uppercase hex, and Base64 | `input` (string, **required**), `encoding` (`"utf8" \| "base64" \| "hex"`, default `"utf8"`), `algorithms` (array of `"md5" \| "sha1" \| "sha256" \| "sha512"`, default all 4), `hmacKey` (optional UTF-8 string) |

### Direct `curl` Invocation Example
```bash
curl -fsSL https://usefulshelf.com/api/mcp \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "cron_explain",
      "arguments": {
        "expression": "*/15 9-18 * * 1-5",
        "count": 3
      }
    }
  }'
```

---

## 3. Batch MCP Endpoint (`/api/mcp/batch` · Up to 50 Items · HTTP 402)

Use `/api/mcp/batch` to execute **1 to 50 invocations** of any of the 5 `/api/mcp` tools in a single HTTP request (max body `512 KiB`).

- **Discovery (`GET https://usefulshelf.com/api/mcp/batch`)**:
  - With `Accept: application/json`: returns `{ endpoint, method: "POST", feature: "mcp-batch-pro", costCredits: 1, maxItems: 50, supportedTools, paymentRequiredStatus: 402, topUpUrl, authHeader }`.
  - With `Accept: text/html`: redirects (`302`) to `https://usefulshelf.com/en/ai-tools`.
- **Execution (`POST https://usefulshelf.com/api/mcp/batch`)**:
  - **Auth Header**: `Authorization: Bearer <US-... | CW-...>` (or `X-Usefulshelf-Code: <code...>`, or an active browser credit session).
  - **Payload**:
    ```json
    {
      "tool": "hash_digest",
      "items": [
        { "input": "release-v1", "algorithms": ["sha256"] },
        { "input": "release-v2", "algorithms": ["sha256"] }
      ],
      "requestId": "batch_2026_001"
    }
    ```
    - `tool`: one of `json_format_validate`, `cron_explain`, `jwt_decode_inspect`, `utm_link_build`, `hash_digest`.
    - `items`: array of `1..50` argument objects matching that tool's schema.
    - `requestId` (optional): `6..96` chars matching `/^[a-z0-9._:-]{6,96}$/i` for safe idempotent retries.
  - **Success (`200 OK`)**: returns `{ ok: true, feature: "mcp-batch-pro", tool, cost: 1, count, bonusCredits, results }`.
  - **Payment Required (`402`)**: when unauthenticated or credits are exhausted, responds with header `Link: <https://usefulshelf.com/en/ai-tools>; rel="payment"` and JSON `{ "error": { "code": "CDK_REQUIRED", "topUpUrl": "https://usefulshelf.com/en/ai-tools", "authHeader": "Authorization: Bearer <US-... | CW-...>" } }`.

```bash
curl -i https://usefulshelf.com/api/mcp/batch \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <US-... | CW-...>' \
  -d '{
    "tool": "utm_link_build",
    "requestId": "campaign-2026-q4",
    "items": [
      { "url": "https://example.com", "utm_source": "github", "utm_medium": "readme" },
      { "url": "https://example.com/docs", "utm_source": "newsletter", "utm_medium": "email" }
    ]
  }'
```

---

## 4. In-Browser W3C WebMCP Tools (`window.navigator.modelContext` · 12 Tools)

When a browser agent visits any of the 12 pages below, UsefulShelf progressively registers structured tools via `window.navigator.modelContext`. Every WebMCP tool runs **100% locally in browser memory (`zeroUpload: true`)**.

| WebMCP Tool Name | Page Route (ZH / EN) | `readOnlyHint` | Key Parameters |
| --- | --- | --- | --- |
| `json_workbench_format` | `/tools/json-format` · `/en/tools/json-format` | `true` | `input`, `indent` (`2\|4`), `sortKeys`, `repair`, `syncEditor` |
| `receipt_workbench_compose` | `/tools/receipt-generator` · `/en/tools/receipt-generator` | `false` | `templateId`, `storeName`, `currency` (`CNY\|USD\|JPY\|EUR\|GBP\|INR`), `taxRate` (`0..1`), `splitPeople` (`1..100`) |
| `image_privacy_status` | `/tools/image-privacy-scrubber` · `/en/tools/image-privacy-scrubber` | `true` | `mode` (`pixelate\|blur\|blackout`), `blockSize` (`6..48`) |
| `image_compress_queue` | `/tools/image-compress` · `/en/tools/image-compress` | `false` | `preset` (`extreme\|balanced\|low`), `format` (`image/webp\|image/jpeg\|image/png`), `runCompression` |
| `mermaid_diagram_render` | `/tools/mermaid-to-image` · `/en/tools/mermaid-to-image` | `false` | `code`, `theme` (`default\|neutral\|dark\|forest`) |
| `utm_builder_generate` | `/tools/utm-builder` · `/en/tools/utm-builder` | `true` | `url`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`, `syncInputs` |
| `cron_schedule_explain` | `/tools/cron-parser` · `/en/tools/cron-parser` | `true` | `expression`, `count` (`1..20`), `syncInput` |
| `hash_checksum_inspect` | `/tools/hash-calculator` · `/en/tools/hash-calculator` | `true` | `text`, `expectedChecksum` |
| `invoice_reimbursement_audit` | `/tools/invoice-reimbursement` · `/en/tools/invoice-reimbursement` | `false` | `layout` (`2up\|4up`) |
| `exam_id_photo_workbench` | `/tools/exam-id-photo` · `/en/tools/exam-id-photo` | `false` | `mode` (`photo\|watermark`), `presetId` (`civil-service-1in\|passport-small-2in\|standard-2in\|teacher-cert\|postgrad-chsi\|social-security\|us-passport`), `background` (`keep\|white\|blue\|red`) |
| `doc_privacy_diff_workbench` | `/tools/doc-privacy-diff` · `/en/tools/doc-privacy-diff` | `false` | `mode` (`scrub\|diff`), `demo` (`dirty-docx\|dirty-xlsx\|contract-pair`) |
| `pdf_vault_workbench` | `/tools/pdf-vault` · `/en/tools/pdf-vault` | `false` | `mode` (`lock\|unlock\|redact`), `demo` (`unprotected\|encrypted-123456\|fake-redacted\|cid-failsafe`) |

---

## 5. Quick Reference: All 60 Live Bilingual Tools

All routes work on both `https://usefulshelf.com/tools/<slug>` (ZH) and `https://usefulshelf.com/en/tools/<slug>` (EN). Unless tagged `[network]` (live external query, zero input storage) or `[local-model]` (browser-local OCR model), every tool is **100% browser-local zero-upload**.

### Flagship Zero-Upload Workbenches

| Slug | English / Chinese Title | ZH & EN Routes | Core Capability |
| --- | --- | --- | --- |
| `exam-id-photo` | Exam ID Photo KB-Fitter & ID Watermark / 考公报名照卡KB与证件防盗签章 | `/tools/exam-id-photo` · `/en/tools/exam-id-photo` | 7 official exam presets, exact KB targeting, solid background swap, 300 DPI + 0 EXIF verify, A4 ID anti-theft watermark |
| `doc-privacy-diff` | Office Doc Metadata Scrubber & Contract Diff / 文档修订擦除与合同防偷改对比 | `/tools/doc-privacy-diff` · `/en/tools/doc-privacy-diff` | Scrub author/comments/revisions/hidden sheets in DOCX/XLSX/PPTX/PDF with post-clean scan + 2-version contract diff |
| `pdf-vault` | PDF AES-256 Lock, Unlock & Redaction X-Ray / PDF密码锁与假涂黑体检 | `/tools/pdf-vault` · `/en/tools/pdf-vault` | PDF 2.0 AES-256 encryption, legitimate known-password unlock (no brute-force), fake black-box redaction audit & physical scrub |
| `invoice-reimbursement` | Invoice Dedupe & A4 Sheet Merger / 电子发票查重与A4合单打印 | `/tools/invoice-reimbursement` · `/en/tools/invoice-reimbursement` | Local PDF/XML/QR invoice parsing, duplicate blocking, A4 2-up/4-up print PDF & CSV ledger export |
| `subscription-audit` | Subscription Renewal Audit & Bill Receipt / 自动续费防坑体检器 | `/tools/subscription-audit` · `/en/tools/subscription-audit` | Local IndexedDB subscription audit, multi-currency totals, RFC 5545 `.ics` cancel reminders & thermal receipt summary |
| `receipt-generator` | Receipt Generator / 消费小票生成器 | `/tools/receipt-generator` · `/en/tools/receipt-generator` | Multi-currency thermal receipt designer, tax & bill split, PNG export & printable layout |
| `qr-code-generator` | QR Code Generator / 二维码生成 | `/tools/qr-code-generator` · `/en/tools/qr-code-generator` | Permanent local static QR codes (URL, Wi-Fi, vCard, menu), scan readability check, PNG/SVG/A5 print standee export |
| `image-privacy-scrubber` | Image EXIF Scrubber & Redaction / 图片去EXIF与打码 | `/tools/image-privacy-scrubber` · `/en/tools/image-privacy-scrubber` | Inspect & strip EXIF/GPS/camera metadata + box pixelate, blur, or blackout redaction |

### Complete 60-Tool Directory by Category

- **Developer Tools (15)**:
  - `json-format` — JSON Formatter Workbench / JSON格式化工作台 (`/tools/json-format` · `/en/tools/json-format`)
  - `code-format` — Code Formatter / 代码格式化 (`/tools/code-format` · `/en/tools/code-format`)
  - `jwt-parser` — JWT Parser / JWT解析 (`/tools/jwt-parser` · `/en/tools/jwt-parser`)
  - `cron-parser` — Cron Parser / Cron解析器 (`/tools/cron-parser` · `/en/tools/cron-parser`)
  - `hash-calculator` — Hash Calculator / 哈希计算 (`/tools/hash-calculator` · `/en/tools/hash-calculator`)
  - `regex-tester` — Regex Tester / 正则测试 (`/tools/regex-tester` · `/en/tools/regex-tester`)
  - `base64-converter` — Base64 Converter / Base64转换 (`/tools/base64-converter` · `/en/tools/base64-converter`)
  - `url-encoder` — URL Encoder / URL编码 (`/tools/url-encoder` · `/en/tools/url-encoder`)
  - `url-parser` — URL Parser / URL解析器 (`/tools/url-parser` · `/en/tools/url-parser`)
  - `timestamp-converter` — Timestamp Converter / 时间戳转换 (`/tools/timestamp-converter` · `/en/tools/timestamp-converter`)
  - `uuid-generator` — UUID Generator / UUID生成器 (`/tools/uuid-generator` · `/en/tools/uuid-generator`)
  - `password-generator` — Password Generator / 密码生成器 (`/tools/password-generator` · `/en/tools/password-generator`)
  - `random-string-generator` — Random String / 随机字符串 (`/tools/random-string-generator` · `/en/tools/random-string-generator`)
  - `html-entity-converter` — HTML Entity Converter / HTML实体转换 (`/tools/html-entity-converter` · `/en/tools/html-entity-converter`)
  - `text-diff` — Text Diff / 文本对比 (`/tools/text-diff` · `/en/tools/text-diff`)

- **Document Tools (7)**:
  - `doc-privacy-diff` — Office Doc Metadata Scrubber & Contract Diff / 文档修订擦除与合同防偷改对比 (`/tools/doc-privacy-diff` · `/en/tools/doc-privacy-diff`)
  - `pdf-vault` — PDF AES-256 Lock, Unlock & Redaction X-Ray / PDF密码锁与假涂黑体检 (`/tools/pdf-vault` · `/en/tools/pdf-vault`)
  - `invoice-reimbursement` — Invoice Dedupe & A4 Sheet Merger / 电子发票查重与A4合单打印 (`/tools/invoice-reimbursement` · `/en/tools/invoice-reimbursement`)
  - `pdf-merge` — PDF Merge / PDF合并 (`/tools/pdf-merge` · `/en/tools/pdf-merge`)
  - `pdf-converter` — Image to PDF / 图片转PDF (`/tools/pdf-converter` · `/en/tools/pdf-converter`)
  - `markdown-preview` — Markdown Preview / Markdown预览 (`/tools/markdown-preview` · `/en/tools/markdown-preview`)
  - `mermaid-to-image` — Mermaid to PNG/SVG / Mermaid 转 PNG / SVG (`/tools/mermaid-to-image` · `/en/tools/mermaid-to-image`)

- **Image Tools (7)**:
  - `exam-id-photo` — Exam ID Photo KB-Fitter & ID Watermark / 考公报名照卡KB与证件防盗签章 (`/tools/exam-id-photo` · `/en/tools/exam-id-photo`)
  - `image-privacy-scrubber` — Image EXIF Scrubber & Redaction / 图片去EXIF与打码 (`/tools/image-privacy-scrubber` · `/en/tools/image-privacy-scrubber`)
  - `image-compress` — Image Compressor / 图片压缩 (`/tools/image-compress` · `/en/tools/image-compress`)
  - `image-resize-crop` — Image Resizer / 图片尺寸调整 (`/tools/image-resize-crop` · `/en/tools/image-resize-crop`)
  - `image-convert` — Image Format Converter / 图片格式转换 (`/tools/image-convert` · `/en/tools/image-convert`)
  - `image-watermark` — Image Watermark / 图片水印 (`/tools/image-watermark` · `/en/tools/image-watermark`)
  - `ocr` — OCR Text Recognition / OCR文字识别 `[local-model]` (`/tools/ocr` · `/en/tools/ocr`)

- **Life Tools (10)**:
  - `qr-code-generator` — QR Code Generator / 二维码生成 (`/tools/qr-code-generator` · `/en/tools/qr-code-generator`)
  - `qr-code-parser` — QR Code Parser / 二维码解析 (`/tools/qr-code-parser` · `/en/tools/qr-code-parser`)
  - `receipt-generator` — Receipt Generator / 消费小票生成器 (`/tools/receipt-generator` · `/en/tools/receipt-generator`)
  - `subscription-audit` — Subscription Renewal Audit & Bill Receipt / 自动续费防坑体检器 (`/tools/subscription-audit` · `/en/tools/subscription-audit`)
  - `barcode-generator` — Barcode Generator / 条形码生成 (`/tools/barcode-generator` · `/en/tools/barcode-generator`)
  - `calculator` — Online Calculator / 在线计算器 (`/tools/calculator` · `/en/tools/calculator`)
  - `percentage-calculator` — Percentage Calculator / 百分比计算器 (`/tools/percentage-calculator` · `/en/tools/percentage-calculator`)
  - `bmi-calculator` — BMI Calculator / BMI计算器 (`/tools/bmi-calculator` · `/en/tools/bmi-calculator`)
  - `loan-calculator` — Loan Calculator / 贷款计算器 (`/tools/loan-calculator` · `/en/tools/loan-calculator`)
  - `weather-query` — Weather Query / 天气查询 `[network]` (`/tools/weather-query` · `/en/tools/weather-query`)

- **Productivity Tools (10)**:
  - `todo-list` — Todo List / 待办清单 (`/tools/todo-list` · `/en/tools/todo-list`)
  - `schedule-planner` — Schedule Planner / 日程规划 (`/tools/schedule-planner` · `/en/tools/schedule-planner`)
  - `pomodoro-timer` — Pomodoro Timer / 番茄钟 (`/tools/pomodoro-timer` · `/en/tools/pomodoro-timer`)
  - `date-calculator` — Date Calculator / 日期计算器 (`/tools/date-calculator` · `/en/tools/date-calculator`)
  - `word-counter` — Word Counter / 字数统计 (`/tools/word-counter` · `/en/tools/word-counter`)
  - `text-process` — Text Processing / 文本处理 (`/tools/text-process` · `/en/tools/text-process`)
  - `case-converter` — Case Converter / 大小写转换 (`/tools/case-converter` · `/en/tools/case-converter`)
  - `lorem-generator` — Lorem Generator / Lorem生成 (`/tools/lorem-generator` · `/en/tools/lorem-generator`)
  - `batch-rename` — Batch Rename List / 文件重命名清单 (`/tools/batch-rename` · `/en/tools/batch-rename`)
  - `translate` — Translation Tool / 翻译工具 `[network]` (`/tools/translate` · `/en/tools/translate`)

- **Design Tools (3)**:
  - `color-picker` — Color Picker / 颜色选择器 (`/tools/color-picker` · `/en/tools/color-picker`)
  - `color-contrast-checker` — Color Contrast Checker / 颜色对比度 (`/tools/color-contrast-checker` · `/en/tools/color-contrast-checker`)
  - `css-gradient-generator` — CSS Gradient Generator / CSS渐变生成器 (`/tools/css-gradient-generator` · `/en/tools/css-gradient-generator`)

- **Conversion Tools (3)**:
  - `unit-converter` — Unit Converter / 单位换算 (`/tools/unit-converter` · `/en/tools/unit-converter`)
  - `radix-converter` — Base Converter / 进制转换 (`/tools/radix-converter` · `/en/tools/radix-converter`)
  - `currency-converter` — Currency Exchange / 汇率换算 `[network]` (`/tools/currency-converter` · `/en/tools/currency-converter`)

- **Site Owner Tools (3)**:
  - `device-check` — Device Performance Check / 设备性能检测 (`/tools/device-check` · `/en/tools/device-check`)
  - `ip-reputation` — IP Reputation Check / IP纯净度检测 `[network]` (`/tools/ip-reputation` · `/en/tools/ip-reputation`)
  - `utm-builder` — UTM Builder / UTM链接生成器 (`/tools/utm-builder` · `/en/tools/utm-builder`)

- **Data Tools (2)**:
  - `csv-json-converter` — CSV to JSON / CSV转JSON (`/tools/csv-json-converter` · `/en/tools/csv-json-converter`)
  - `yaml-converter` — YAML Converter / YAML转换 (`/tools/yaml-converter` · `/en/tools/yaml-converter`)

---

## 6. Privacy & Security Boundary

- All `/api/mcp` tools are pure stateless functions (`readOnlyHint: true`, zero database writes, `Cache-Control: no-store`).
- Never transmit production signing private keys, live credentials, or confidential user documents to remote HTTP endpoints. `jwt_decode_inspect` inspects public header/payload claims only; use the browser-local zero-upload workbenches (`/tools/jwt-parser`, `/tools/pdf-vault`, `/tools/doc-privacy-diff`, `/tools/image-privacy-scrubber`) whenever processing sensitive files.
- Network-assisted utilities (`weather-query`, `translate`, `currency-converter`, `ip-reputation`) are explicitly marked and do not store user query history.
