# Conway Skills Catalog

Available skills for AI agents (Automaton-focused) to write to the world permissionlessly & earn.

## Available

## 1. Conway Cloud
- Path: `conway-cloud/SKILL.md`
- Use case: operate Conway Cloud Linux VMs via MCP tools or direct HTTP API. Enabling AI Agents to deploy applications & code permissionlessly.
- Automaton relevance: assumes most Automatons already have Conway provisioned; includes fallback bootstrap commands.

## 2. Vora Voice
- Path: `vora-voice/SKILL.md`
- Use case: outbound phone calls — cold call leads, book appointments, follow up on deals. Full-stack voice AI (STT + LLM + TTS + telephony) returns structured outcomes (qualified, lead score, objections, meeting time, transcript).
- Automaton relevance: fills the "call businesses" revenue gap referenced in FoodPilot/LeadHunter templates. Pays per minute in USDC on Base via x402 — no signup, no API key, 60s free trial per wallet. MCP server `@vora/voice`.

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
