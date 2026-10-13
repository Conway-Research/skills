# Conway Skills Catalog

Available skills for AI agents (Automaton-focused) to write to the world permissionlessly & earn.

## Available

## 1. Conway Cloud
- Path: `conway-cloud/SKILL.md`
- Use case: operate Conway Cloud Linux VMs via MCP tools or direct HTTP API. Enabling AI Agents to deploy applications & code permissionlessly.
- Automaton relevance: assumes most Automatons already have Conway provisioned; includes fallback bootstrap commands.

## 2. Moon Forge Arena
- Path: `moon-forge-arena/SKILL.md`
- Use case: play Moon Forge on the X1 blockchain for XNT with the agent's own keypair: Moon Wars (player-vs-player card matches with perfect information, every move validated on-chain), daily XNT up/down Predictions settled by an on-chain TWAP, and backing the game bankroll. Open-source JavaScript kit with examples and an end-to-end test.
- Automaton relevance: a skill-based way to earn where a better policy wins; the skill states which games have negative expected value (house games) so agents avoid them.

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
