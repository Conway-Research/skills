---
name: base-wallet-audit
description: "Reconcile an automaton's own Base wallet against on-chain truth: USDC Transfer events on both legs, gas per leg, book-vs-chain delta, and verdict ruler recording. Teaches agents the accounting errors that make self-reported P&L lie."
auto-activate: true
triggers: [audit, reconcile, wallet, pnl, ledger, "profit", "loss"]
---

# Base Wallet Audit (proof-of-agency discipline)

## Why this skill exists
A public audit of automaton P&L claims found the most common failure is not strategy — it is accounting that flatters itself (e.g., counting the sell leg's gas but silently dropping the buy leg, overstating profit by 53% on a single position). Your database is a model of what happened. The chain is what happened. This skill reconciles the two.

## Procedure

### 1. Pull ground truth (never trust your book first)
- List all USDC `Transfer` events to AND from your wallet on Base (RPC: `https://mainnet.base.org`).
- For every transaction hash from those events, fetch the receipt and compute true gas:
  `gasUsed × effectiveGasPrice + l1Fee` — for BOTH legs of any swap.

### 2. Rebuild P&L from events only
```
net_usdc = sum(Transfer-in) − sum(Transfer-out)          # excludes gas
gas_total = sum(receipt gas for every tx you signed)
true_pnl = net_usdc − gas_total
wallet_delta = current_balance − balance_at_period_start
```
- Assert: `true_pnl ≈ wallet_delta` (tolerance: dust). If not, your book is wrong — find out why before trading again.

### 3. Record the ruler with every verdict
A verdict like "profitable" is meaningless without the cost ruler it was measured against. Record: pool fee tier, position size, measured round-trip cost in ticks/percent, and the date the ruler was measured. When the ruler changes (new pool, new size), re-open every verdict closed against the old ruler.

### 4. Report format (post this publicly if you claim earnings)
```
period: <start> → <end>
transfers_in:  X USDC (n events)
transfers_out: Y USDC (n events)
gas_total:     G ETH-equivalent (n txs, both legs)
book_pnl:      $A     chain_pnl: $B     delta: $|A−B| (must be ~0)
ruler:         <round-trip cost at your size, measured not assumed>
verdict:       net profitable / not — closed against ruler above
```

## Hard rules
- Never publish an earnings claim without a wallet address the reader can check.
- Count both gas legs. Always. Reconcile against the chain before any claim.
- At small capital, friction dominates: a perfect oracle loses at short horizons. If your edge is smaller than your ruler, the honest verdict is "no edge."
