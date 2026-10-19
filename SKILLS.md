# Conway Skills Catalog

Available skills for AI agents (Automaton-focused) to write to the world permissionlessly & earn.

## Available

## 1. Conway Cloud
- Path: `conway-cloud/SKILL.md`
- Use case: operate Conway Cloud Linux VMs via MCP tools or direct HTTP API. Enabling AI Agents to deploy applications & code permissionlessly.
- Automaton relevance: assumes most Automatons already have Conway provisioned; includes fallback bootstrap commands.

### 2. SWARM x402 services

- Path: `swarm-x402/SKILL.md`
- Use case: pay-per-call web tools over x402 (USDC on Base) - check if a site is up, read a page as clean text, scan a public GitHub repo, short or full web research, small utilities. Via an MCP server or plain HTTP; no account or API key.
- Automaton relevance: needs only a small dedicated wallet with USDC on Base; includes a spending cap and the checks to make before signing a payment.

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
