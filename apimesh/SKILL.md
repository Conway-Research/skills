---
name: apimesh
description: Call 12 web-analysis APIs that accept x402 micropayments — no API keys, no accounts.
auto-activate: false
---

# APIMesh

Use when an agent needs web intelligence: domain availability, SEO scores, security headers, Core Web Vitals, email authentication, redirect chains, brand assets, or indexability checks. Every call is pay-per-use via x402 (USDC on Base).

## Precheck

```bash
# Verify the MCP server is reachable
curl -sf https://apimesh.xyz/.well-known/x402 > /dev/null && echo "APIMesh OK"
```

If using MCP tools, confirm the server is registered:

```bash
npx @mbeato/apimesh-mcp-server
```

## Available tools

| Tool | What it does | Cost |
|------|-------------|------|
| `web_checker` | Check brand availability across 5 TLDs, GitHub, npm, PyPI, Reddit | $0.005 |
| `http_status_checker` | Live HTTP status of any URL | $0.001 |
| `favicon_checker` | Detect favicon URL, format, status | $0.001 |
| `microservice_health_check` | Parallel health check up to 10 URLs | $0.003 |
| `robots_txt_parser` | Parse robots.txt into structured rules | $0.001 |
| `core_web_vitals` | Lighthouse scores + LCP, CLS, INP field data | $0.005 |
| `security_headers` | Audit 10 security headers, graded A+ to F | $0.005 |
| `redirect_chain` | Trace full redirect chain with latency per hop | $0.001 |
| `email_security` | SPF, DMARC, DKIM, MX analysis | $0.010 |
| `seo_audit` | On-page SEO audit with 0-100 score | $0.003 |
| `indexability_checker` | 5-layer search-engine indexability check | $0.001 |
| `brand_assets` | Extract logo, favicon, theme colors, OG image | $0.002 |

Most tools also expose a free `/preview` endpoint with limited data.

## Preferred interface: MCP tools

If the MCP server is configured, call tools by name (e.g. `web_checker`, `seo_audit`). The server handles x402 payment automatically when an x402-capable wallet is available.

## HTTP fallback

Each tool lives at `https://{tool-name}.apimesh.xyz/check` (GET or POST). On first request, the server returns HTTP 402 with payment details. Sign a USDC payment on Base and resend with the `X-PAYMENT` header.

```bash
# Example: check security headers
curl https://security-headers.apimesh.xyz/check?url=https://example.com
# Returns 402 with payment requirements

# After signing payment:
curl -H "X-PAYMENT: <signed-payment>" \
  "https://security-headers.apimesh.xyz/check?url=https://example.com"
```

## Execution pattern

1. Prefer MCP tools over raw HTTP when available.
2. Use free `/preview` endpoints for quick checks before paying.
3. Batch related checks (e.g. `seo_audit` + `core_web_vitals` + `security_headers` for a full site review).
4. Cache results — most web data is stable for minutes to hours.

## Guardrails

- Never send authentication tokens or cookies as URL parameters to any APIMesh endpoint.
- Do not call paid endpoints in tight loops — use preview endpoints for iteration.
- Costs are sub-cent per call but can accumulate; confirm with the user before running bulk audits (10+ calls).
