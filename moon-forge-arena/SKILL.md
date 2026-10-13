---
name: moon-forge-arena
description: Play Moon Forge on the X1 blockchain for XNT with your own keypair - Moon Wars (skill-based player-vs-player card matches, every move validated on-chain), daily XNT up/down Predictions, and backing the game bankroll - using the open-source agent kit.
auto-activate: false
triggers: [moon forge, moon wars, x1 blockchain, xnt, xnt prediction, play for xnt]
---
# Moon Forge Arena (X1)

Moon Forge is an open-source protocol on X1 (an SVM chain; native token XNT). Its games are instructions of one program,
`57UE1U1t23ztg2noLp8pcpGW1B1Xw25rLH6ra9Mchea9`: outcomes are decided on-chain and anyone can verify them.
Part of every pot goes to a reward pool that pays people who burn XEN, so playing also funds that pool.

## Where an agent can have an edge (and where it cannot)
- **Moon Wars** - player vs player, perfect information (both decks are public after the start), every move validated
  by the program. A better policy beats a worse one. Winner takes both stakes minus 2% (1% reward pool, 0.5% lottery,
  0.5% architect). Draw after 100 rounds = full refund. **2 minutes per turn: running out of time loses.**
- **Predictions** - one round per UTC day: entries for the first 12 h, start price = 15-minute TWAP of the XDEX
  WXNT/USDC.X pool at the lock, end price = the same TWAP 24 h later. Winners share the pot pro-rata minus 2%.
  A tie, an empty side or missing price data refunds everyone. One entry per wallet per round; pot capped at 200 XNT.
- **Be the House** - deposit XNT into the game bankroll and share its result (0.5% of every house stake stays in it,
  it pays the winners). 24 h withdrawal cooldown. You can lose.
- **House games (Coin Flip, High-Low, Void Rush)** have a fixed 2% edge: negative expected value. Do not use them as an
  earning strategy.

## Requirements
- Node 20+, an X1 keypair you control (never share its secret), XNT on X1 for fees and stakes.
  XNT can be obtained by bridging USDC from Solana at https://app.bridge.x1.xyz and swapping on https://app.xdex.xyz.
- A live Moon Forge Artifact is the Moon Wars ticket: a Lunar Dust costs 10 XNT, 5 XNT stay inside it as a floor
  you get back by recycling it.
- RPC: https://rpc.mainnet.x1.xyz

## Setup
```bash
git clone https://github.com/xen-moon-forge-protocol/Moon-Forge
cd Moon-Forge/agents/kit && npm install
AGENT_KEY=/path/to/keypair.json node examples/status.mjs
```

## Workflows
1. **Decide** - `mf.status()` returns wallet and game balances, the reward pool, the XNT price (on-chain TWAP),
   open Moon Wars matches and whether today's prediction round still takes entries.
2. **Moon Wars** - `mf.forgeLunarDust()` once, `mf.deposit(x)`, then `mf.joinWar(war)` on an open match (or
   `mf.createWar(stake)`), `mf.playMatch(war, { policy })` and `mf.settleWar(war)`. Write your own `policy(w)` that
   returns one of `mf.legalMoves(w)`; `mf.greedyMove` is a simple baseline. Unjoined match: `mf.cancelWar(war)` refunds.
3. **Predictions** - `mf.enterPrediction('up' | 'down', x)` while entries are open; later `mf.claimPredictions()`.
   Locking and settling are permissionless (`mf.advancePrediction(id)`), the public keeper usually does them.
4. **Be the House** - `mf.houseQuote(x)`, then `mf.houseDeposit(x)` (1% slippage guard).
See `examples/war-bot.mjs` and `examples/predict.mjs` for complete loops.

## Guardrails
- Never print, log or send your private key; the kit only signs locally.
- Keep a reserve of at least 0.05 XNT in the wallet for fees.
- Stake only what you can afford to lose. Losing a match or a round loses the stake.
- Keep polling during a match: a missed 2-minute turn loses it.
- Only the wallet owner can enter P2P games (bot session keys cannot).
- The program is upgradeable during its public test period and has no third-party audit (internal reviews are public).

## Verify before trusting
- Source, tests and rules: https://github.com/xen-moon-forge-protocol/Moon-Forge
- Live numbers and checklist: https://xen-moon-forge-protocol.github.io/Moon-Forge/start
