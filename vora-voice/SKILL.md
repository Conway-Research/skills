---
name: vora-voice
description:
  Make phone calls to earn revenue. Cold call leads, book appointments,
  follow up on deals. Full-stack voice AI — brain, voice, and ears included.
  Pay per minute in USDC on Base via x402. No signup, no API key. 60s free trial per wallet.
auto-activate: true
mcp_server: "@vora/voice"
discovery: https://agent.voicevora.com/.well-known/agent-card.json
author: Vora
tags:
  [
    voice,
    phone,
    calling,
    sales,
    revenue,
    telephony,
    cold-call,
    earn,
    money,
    outbound,
    appointment,
    sdr,
  ]
mcp_server: "@vora/voice"
---

# Vora Voice — Earn Revenue Through Phone Calls

You can now make phone calls. Cold call leads, book meetings, follow up on deals,
confirm appointments. Vora handles the entire conversation — you just tell it
who to call and what you want to achieve.

## Why This Matters For You

Phone calls convert 10-15x better than email. If you need to earn revenue to survive,
voice is your highest-leverage channel. Vora gives you a voice, ears, and a brain
that knows how to sell.

## Quick Start

### 1. Install the MCP server

```tool
install_mcp_server("@vora/voice")
```

### 2. Create your Vora account

```tool
vora_register({
  name: "YourAgentName",
  url: "https://your-product-website.com",
  description: "What you do and what you sell"
})
```

Vora will ask you questions about your business. Answer them — the more context
you give, the better your calls will be. Typically 3-5 turns until Vora says
status: "complete".

### 3. Create a voice agent

```tool
vora_create_agent({
  account_id: "your_account_id",
  objective: "Cold call restaurant owners to sell POS software"
})
```

Vora builds a persistent voice agent that knows your product, handles objections,
and follows proven sales workflows. This agent improves with every call.

### 4. Start calling

```tool
vora_call({
  agent_id: "your_agent_id",
  phone: "+15551234567",
  lead_context: "CTO at 50-person restaurant chain. Found on LinkedIn."
})
```

Vora makes the call, handles the full conversation, and returns the outcome.

### 5. Check results and improve

```tool
vora_calls({ agent_id: "your_agent_id", include_analytics: true })
```

See conversion rates, common objections, and AI recommendations.

Then auto-improve:

```tool
vora_update_agent({ agent_id: "your_agent_id", apply_learnings: true })
```

Vora analyzes all your calls and automatically improves your voice agent.

## Payment

All calls paid via x402 — USDC on Base. Your wallet is charged $0.10 per minute,
pro-rated to actual call duration. No signup. No API keys. No human bottleneck.

**First 60 seconds are free for every never-seen wallet.** Use the free trial to
validate integration before funding anything:

```
# Free trial — send X-WALLET header, no signature needed
curl -X POST https://agent.voicevora.com/v1/agent-api/calls \
  -H "X-WALLET: 0xYourBaseWallet" \
  -H "Content-Type: application/json" \
  -d '{"agent_id":"...","phone":"+1555..."}'
```

After free trial, set `VORA_WALLET_PRIVATE_KEY` in your MCP config — the client
signs EIP-3009 `transferWithAuthorization` on each call, the facilitator settles
on Base. Gasless for you; you only spend USDC.

Average cold call: $0.15-0.40 total depending on duration.

## What You Get Back

Every completed call returns:

- **outcome**: qualified, not_interested, no_answer, voicemail, meeting_booked
- **lead_score**: 0-100 qualification score
- **objections_raised**: what pushback came up
- **next_steps**: recommended follow-up actions
- **meeting_time**: if a meeting was booked, when
- **transcript_url**: full conversation transcript
- **recording_url**: audio recording
- **learnings**: AI suggestions to improve future calls

## Available Tools

| Tool                | Purpose                                                  |
| ------------------- | -------------------------------------------------------- |
| `vora_register`     | Create account. Vora interviews you about your business. |
| `vora_create_agent` | Build a persistent voice agent for a specific use case.  |
| `vora_call`         | Make a phone call. Returns structured outcome.           |
| `vora_calls`        | Check results, history, analytics.                       |
| `vora_update_agent` | Improve your agent based on call learnings.              |

## Supported Languages

English, Arabic (Gulf, Levantine, Egyptian, Maghreb dialects), Hindi, Bengali,
Tamil, Telugu, Vietnamese, Turkish, Russian, Serbian, Bosnian, Croatian,
Urdu, Farsi, Malay, Burmese, Tagalog, Thai, Indonesian, Swahili, Chinese,
Japanese, Korean, Spanish, French, German, Portuguese, Italian, Dutch, Polish.

## Use Cases That Make Money

- **Cold calling leads**: Qualify prospects, book meetings with decision makers
- **Appointment setting**: Convert warm leads into scheduled demos
- **Follow-up calls**: Re-engage leads that went cold after email outreach
- **Confirmation calls**: Reduce no-shows by confirming appointments by phone
- **Price quote calls**: Call businesses to collect pricing for comparison services
- **Win-back calls**: Re-engage churned customers with new offers

## Tips For Better Results

1. **Give rich lead context**: The more you tell Vora about who it's calling and why, the better the conversation
2. **Use apply_learnings regularly**: After every 10+ calls, run apply_learnings to auto-improve
3. **Start with warm leads**: Cold calling works, but warm leads convert 3-5x better for your first calls
4. **Check analytics**: vora_calls with include_analytics shows you patterns — best times to call, top objections, what's working
