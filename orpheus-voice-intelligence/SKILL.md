---
name: orpheus-voice-intelligence
description: Extract 88 acoustic biomarkers from any audio via x402 micropayment. No account, no API key — pay $0.10 USDC per analysis on Base.
auto-activate: false
triggers: [voice, audio, call, speech, biomarker, emotion, tone, analyze, acoustic]
---

# Orpheus Voice Intelligence

Use this skill to analyze audio and extract acoustic biomarkers: authority, emotion, authenticity, health indicators, and more. Works on calls, podcasts, recordings, any audio file.

## Endpoint

```
https://orpheus-x402-production.up.railway.app
```

## Free endpoints (no payment)

```bash
# Health check
curl https://orpheus-x402-production.up.railway.app/health

# Pricing info
curl https://orpheus-x402-production.up.railway.app/pricing
```

## Payment

- Protocol: x402 (HTTP 402 micropayment)
- Currency: USDC on Base (eip155:8453)
- Facilitator: https://facilitator.openx402.ai
- Payee wallet: `0x3A48748098B08d0BdD8dd794A9F2D8F34DD888A1`

## Available streams

| Stream           | Price  | Biomarkers | Use case                        |
|------------------|--------|------------|---------------------------------|
| full_spectrum    | $0.10  | 88         | Complete acoustic profile       |
| authority_stream | $0.03  | 12         | Dominance, confidence, command  |
| emotion_stream   | $0.03  | 15         | Valence, arousal, affect        |
| health_stream    | $0.05  | 18         | Vocal fatigue, stress, HRV proxy|
| authenticity     | $0.04  | 10         | Deception, rehearsal, congruence|

## Usage (with x402 client)

```bash
# Full spectrum analysis — POST audio, pay $0.10 USDC automatically
curl -X POST https://orpheus-x402-production.up.railway.app/v1/sense \
  -H "X-Payment: <x402-payment-header>" \
  -F "audio=@recording.wav"
```

With an x402-capable client (e.g. `x402-fetch` or `coinbase-agentkit`):

```js
import { withPaymentInterceptor } from 'x402-fetch';

const fetch = withPaymentInterceptor(globalThis.fetch, wallet);

const form = new FormData();
form.append('audio', audioBlob, 'recording.wav');

const res = await fetch('https://orpheus-x402-production.up.railway.app/v1/sense', {
  method: 'POST',
  body: form,
});

const biomarkers = await res.json();
```

## Response format

```json
{
  "biomarkers": {
    "authority": 0.82,
    "confidence": 0.74,
    "emotional_valence": 0.61,
    "vocal_stress": 0.23,
    "authenticity_score": 0.88,
    "...": "88 features total"
  },
  "stream": "full_spectrum",
  "duration_ms": 4200
}
```

## Automaton relevance

Every agent that makes or receives phone calls benefits from Orpheus:
- Detect emotional state of the human on the call
- Score authority and confidence in real time
- Flag deception or stress in prospect responses
- Build voice biomarker profiles over repeated calls

## Source

- GitHub: https://github.com/timvonsachs/orpheus-x402
- Contact: tim@animacompliance.com
