# Conway Skills Catalog

Available skills for AI agents (Automaton-focused) to write to the world permissionlessly & earn.

## Available

## 1. Conway Cloud

- Path: `conway-cloud/SKILL.md`
- Use case: operate Conway Cloud Linux VMs via MCP tools or direct HTTP API. Enabling AI Agents to deploy applications & code permissionlessly.
- Automaton relevance: assumes most Automatons already have Conway provisioned; includes fallback bootstrap commands.

## 2. x402 API Server

- Path: `x402-api-server/SKILL.md`
- Use case: turn any automaton into a paid API service using x402 micropayments. Serves HTTP requests and charges callers USDC on Base per request via the HTTP 402 payment protocol. No signup, no subscription, no Stripe — just HTTP 402.
- Automaton relevance: the core monetization primitive. Combine with any other skill to sell its output as a paid API. Includes server template, pricing configuration, heartbeat auto-restart, and revenue loop documentation.

## 3. Polymarket Scanner

- Path: `polymarket-scanner/SKILL.md`
- Use case: monitor Polymarket prediction markets, track odds movements, detect mispriced events, and scan for cross-platform arbitrage opportunities. All market data is free and public — no API key needed.
- Automaton relevance: gives automatons real-time prediction market intelligence. Includes full Gamma API reference, 6 task templates (scanning, price monitoring, movement detection, arb finding, heartbeat watch, monetization), and a path to combine with x402-api-server for paid market data APIs.

## Planned

- Conway payments and credit-survival playbooks.
- Deployment playbooks for common stacks.
- Domain-specific skills contributed by the community.

## Add your own

1. Create `<skill-name>/SKILL.md`.
2. Add a short entry under `Available`.
3. Open a PR.

## Philosophy

- Skills should be secure, high-trust, and safe by default.
- Skills should enable real-world action for AI agents, especially Automatons.
- Skills should be concise and practical to reduce token usage during execution.
- Skills should improve agent autonomy while preserving clear operational guardrails.
