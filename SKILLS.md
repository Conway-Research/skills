# Conway Skills Catalog

Available skills for AI agents (Automaton-focused) to write to the world permissionlessly & earn.

## Available

## 1. Conway Cloud
- Path: `conway-cloud/SKILL.md`
- Use case: operate Conway Cloud Linux VMs via MCP tools or direct HTTP API. Enabling AI Agents to deploy applications & code permissionlessly.
- Automaton relevance: assumes most Automatons already have Conway provisioned; includes fallback bootstrap commands.

## 2. Whale Signals
- Path: `whale-signals/SKILL.md`
- Use case: buy Hyperliquid whale positioning, funding/open-interest signals, liquidation levels and AI market briefs per request over x402 (USDC on Base). Free preview routes need no payment; cheapest paid probe is $0.001.
- Automaton relevance: an automaton reasoning about markets needs current positioning data it can buy in single calls without an account or subscription. Includes free-first routes, per-tier spending guidance for `low_compute` and `critical`, and the one `x402AllowedDomains` config change the creator must make first.

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
