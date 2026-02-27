---
name: conway-cloud
description: Operate Conway Cloud from AI agents, with Automaton-first defaults and HTTP fallback.
auto-activate: true
---

# Conway Cloud Operations

Use this skill for sandbox lifecycle, command execution, file operations, and service exposure on Conway Cloud.

## Precheck (Automaton-first)

Automatons are usually provisioned already. Validate first, then operate:

```bash
conway-terminal --version
curl https://api.conway.tech/health
```

If `CONWAY_API_KEY` is missing or invalid, use the fallback bootstrap path below.

## Preferred interface: MCP tools

Use MCP tools first when available:

- `sandbox_list`
- `sandbox_create`
- `sandbox_exec`
- `sandbox_read_file`
- `sandbox_write_file`
- `sandbox_expose_port`
- `sandbox_get_url`
- `sandbox_delete`
- Interactive: `sandbox_pty_create`, `sandbox_pty_write`, `sandbox_pty_read`, `sandbox_pty_close`

## HTTP fallback (direct API)

If MCP is unavailable, use direct HTTP calls.

Base URL:

```bash
https://api.conway.tech/v1
```

Auth header on every request:

```bash
Authorization: Bearer $CONWAY_API_KEY
```

Core calls:

1. List sandboxes:
```bash
curl https://api.conway.tech/v1/sandboxes \
  -H "Authorization: Bearer $CONWAY_API_KEY"
```
2. Create sandbox:
```bash
curl -X POST https://api.conway.tech/v1/sandboxes \
  -H "Authorization: Bearer $CONWAY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"task-sandbox","vcpu":1,"memory_mb":512,"disk_gb":2,"region":"us-east"}'
```
3. Run command:
```bash
curl -X POST https://api.conway.tech/v1/sandboxes/<SANDBOX_ID>/exec \
  -H "Authorization: Bearer $CONWAY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command":"pwd && ls -la"}'
```
4. Upload file:
```bash
curl -X POST "https://api.conway.tech/v1/sandboxes/<SANDBOX_ID>/files" \
  -H "Authorization: Bearer $CONWAY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path":"/home/ubuntu/app.js","content":"console.log(\"hello\")"}'
```
5. Download/read file:
```bash
curl "https://api.conway.tech/v1/sandboxes/<SANDBOX_ID>/files?path=/home/ubuntu/app.js" \
  -H "Authorization: Bearer $CONWAY_API_KEY"
```
6. Expose port:
```bash
curl -X POST "https://api.conway.tech/v1/sandboxes/<SANDBOX_ID>/ports?port=3000" \
  -H "Authorization: Bearer $CONWAY_API_KEY"
```
7. Delete sandbox:
```bash
curl -X DELETE "https://api.conway.tech/v1/sandboxes/<SANDBOX_ID>" \
  -H "Authorization: Bearer $CONWAY_API_KEY"
```

## Fallback bootstrap (when not provisioned)

Prefer one-line provisioning:

```bash
curl -fsSL https://conway.tech/terminal.sh | sh
```

Manual install (if API key already exists):

```bash
npm install -g conway-terminal
export CONWAY_API_KEY="cnwy_k_..."
```

Run modes:

- MCP stdio mode: `conway-terminal`
- HTTP mode: `conway-terminal --http 3000`
- HTTP tool check: `curl http://localhost:3000/mcp/v1/tools`

## Execution pattern

1. Reuse existing sandbox when possible.
2. Create minimal resources first; scale up only if needed.
3. Execute in small, verifiable steps.
4. Expose ports only after health checks pass.
5. Tear down idle sandboxes.

## Guardrails

- Never leak `CONWAY_API_KEY`, wallet keys, or tokenized terminal URLs.
- Avoid destructive actions unless explicitly required.
- Keep operations resource-aware and cost-conscious.
