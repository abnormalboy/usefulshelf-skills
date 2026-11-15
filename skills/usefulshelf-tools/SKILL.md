---
name: usefulshelf-tools
description: Stateless deterministic developer utilities from UsefulShelf (usefulshelf.com) via MCP Streamable HTTP (/api/mcp) or JSON-RPC CLI calls — JSON validate/format/repair, 5-field Cron schedule explainer, JWT claim & expiry inspector, UTM campaign URL builder, and MD5/SHA/HMAC digest calculator.
---

# UsefulShelf Deterministic Developer Tools (`usefulshelf-tools`)

Use this skill whenever an agent or developer workflow needs **100% deterministic, zero-hallucination developer data transformations** without spending LLM tokens on arithmetic, calendar math, Base64URL decoding, or cryptographic hashing.

- **Stateless MCP Endpoint**: `https://usefulshelf.com/api/mcp` (JSON-RPC 2.0 over Streamable HTTP, no API key required, zero input persistence)
- **Skills Index Manifest**: `https://usefulshelf.com/api/skills/manifest`
- **Web Interactive Suite**: `https://usefulshelf.com/tools`

## 1. One-Command MCP Setup

### Claude Code / CLI
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

### Install Skill Markdown Locally
```bash
mkdir -p .agents/skills/usefulshelf-tools
curl -fsSL https://usefulshelf.com/skills/usefulshelf-tools/SKILL.md \
  -o .agents/skills/usefulshelf-tools/SKILL.md
```

## 2. Available Deterministic MCP Tools

| Tool Name | Purpose | Key Parameters |
| --- | --- | --- |
| `json_format_validate` | Validate JSON with exact `line`/`column` error locations, format/minify, sort keys, or repair comments/trailing commas | `input` (string), `indent` (`2\|4`), `sortKeys` (bool), `compact` (bool), `repair` (bool) |
| `cron_explain` | Parse 5-field cron expressions (`min hour dom mon dow`) and compute exact upcoming ISO-8601 run timestamps | `expression` (string), `count` (`1..20`), `afterIso` (optional ISO string) |
| `jwt_decode_inspect` | Decode JWT header & payload claims, convert `iat`/`nbf`/`exp` to ISO-8601, and flag expired or unsigned (`alg=none`) tokens | `token` (string), `nowEpochSeconds` (optional number) |
| `utm_link_build` | Normalize HTTP/HTTPS campaign URLs with `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content` | `url` (string), `utm_*` (strings), `extraParams` (object) |
| `hash_digest` | Compute MD5, SHA-1, SHA-256, and SHA-512 checksums or HMAC digests in hex and Base64 | `input` (string), `encoding` (`utf8\|base64\|hex`), `algorithms` (array), `hmacKey` (optional string) |

## 3. Direct Stateless CLI Invocation (`curl`)

When an MCP client is not attached, invoke any tool directly with `curl`:

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

## 4. Privacy & Security Boundary

- Never send production signing private keys or live secret credentials when inspecting tokens; `jwt_decode_inspect` inspects public header/payload claims only, and local browser equivalents are available offline at `https://usefulshelf.com/tools/jwt-parser`.
- All `/api/mcp` tools are pure stateless functions (`readOnlyHint: true`, zero database writes, `Cache-Control: no-store`).
