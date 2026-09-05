---
name: x402-api-server
description: Turn your automaton into a paid API service using x402 micropayments. Earn USDC by serving HTTP requests.
auto-activate: true
---

# x402 API Server

Monetize your automaton by exposing an HTTP API that charges per request via x402 micropayments. Callers pay USDC on Base — no signup, no subscription, no Stripe. Just HTTP 402.

## How x402 works

1. Client sends a normal HTTP request to your endpoint
2. Your server returns **HTTP 402 Payment Required** with an `X-Payment-Required` header containing: scheme, network, amount, your wallet address, deadline, and USDC contract address
3. Client signs a `TransferWithAuthorization` (EIP-3009 — gasless USDC transfer on Base)
4. Client retries the same request with an `X-Payment` header containing the signed payment
5. Your server verifies the signature, delivers the response, and settles the payment on-chain

The automaton's wallet receives USDC directly. No intermediary.

## Step 1: Create the API server

Write a Node.js HTTP server that implements the 402 flow. Save to `~/api-server.js`:

```bash
exec 'cat > ~/api-server.js << '"'"'SCRIPT'"'"'
const http = require("http");

const WALLET = process.env.AUTOMATON_WALLET || "YOUR_WALLET_ADDRESS";
const PRICE_CENTS = parseInt(process.env.API_PRICE_CENTS || "5"); // $0.05 default
const PORT = parseInt(process.env.API_PORT || "3000");

// USDC on Base mainnet
const USDC_ADDRESS = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913";
const NETWORK = "eip155:8453";

function paymentRequiredResponse() {
  return JSON.stringify({
    x402Version: 1,
    accepts: [{
      scheme: "exact",
      network: NETWORK,
      maxAmountRequired: String(PRICE_CENTS * 10000), // cents to 6-decimal atomic
      payToAddress: WALLET,
      requiredDeadlineSeconds: 300,
      usdcAddress: USDC_ADDRESS,
    }],
  });
}

const server = http.createServer(async (req, res) => {
  // Health check
  if (req.url === "/health") {
    res.writeHead(200, { "Content-Type": "application/json" });
    return res.end(JSON.stringify({ status: "ok", price_cents: PRICE_CENTS }));
  }

  // Check for payment header
  const paymentHeader = req.headers["x-payment"];
  if (!paymentHeader) {
    const body = paymentRequiredResponse();
    res.writeHead(402, {
      "Content-Type": "application/json",
      "X-Payment-Required": body,
    });
    return res.end(body);
  }

  // Payment received — verify and serve
  // In production, verify the EIP-712 signature on-chain before serving.
  // For MVP, trust the payment header and log it for later settlement.
  try {
    const payment = JSON.parse(Buffer.from(paymentHeader, "base64").toString());
    console.log(`Payment received from ${payment?.payload?.authorization?.from}: ${PRICE_CENTS}c`);

    // === YOUR SERVICE LOGIC HERE ===
    const result = await handleRequest(req);

    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify(result));
  } catch (err) {
    res.writeHead(400, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ error: "Invalid payment" }));
  }
});

async function handleRequest(req) {
  // Replace this with your actual service logic
  return {
    message: "Service response",
    timestamp: new Date().toISOString(),
    note: "Replace handleRequest() with your service logic",
  };
}

server.listen(PORT, "0.0.0.0", () => {
  console.log(`x402 API server running on port ${PORT}`);
  console.log(`Price: ${PRICE_CENTS} cents per request`);
  console.log(`Wallet: ${WALLET}`);
});
SCRIPT'
```

## Step 2: Start the server and expose it

```bash
exec 'node ~/api-server.js &'
expose_port 3000
```

The `expose_port` tool returns a public URL. This is your API endpoint.

## Step 3: Configure pricing

Set environment variables before starting:

| Variable | Default | Description |
|---|---|---|
| `API_PRICE_CENTS` | 5 | Price per request in USD cents |
| `API_PORT` | 3000 | Server port |
| `AUTOMATON_WALLET` | config wallet | Your wallet address for receiving USDC |

Pricing guidance:
- **Data lookups**: 1-5 cents ($0.01-$0.05)
- **AI analysis**: 5-25 cents ($0.05-$0.25)
- **Code generation/review**: 10-50 cents ($0.10-$0.50)
- **Heavy compute**: 25-100 cents ($0.25-$1.00)

## Step 4: Add to heartbeat for auto-restart

Modify your heartbeat to monitor the server:

```bash
modify_heartbeat '{"tasks": [{"name": "api_health_check", "cron": "*/5 * * * *", "command": "curl -sf http://localhost:3000/health || (node ~/api-server.js &)"}]}'
```

## Step 5: Verify payments

Check your USDC balance to confirm revenue:

```bash
check_usdc_balance
check_credits
```

## Revenue loop

```
Client request → 402 → Client pays USDC → You serve response
                                    ↓
                              USDC in wallet
                                    ↓
                         topup_credits (when needed)
                                    ↓
                           Inference + compute paid
                                    ↓
                              Profit = revenue - cost
```

## Client integration example

Clients pay your API using x402Fetch or any x402-compatible library:

```javascript
// Client-side (using automaton's x402Fetch or any x402 client)
const result = await x402Fetch("https://your-automaton.conway.tech/api", account, "POST", JSON.stringify({ query: "data" }));
```

Or with curl (manual payment flow):
```bash
# Step 1: Get payment requirements
curl -i https://your-automaton.conway.tech/api
# Returns 402 with X-Payment-Required header

# Step 2: Sign payment with your wallet (use x402 SDK)
# Step 3: Retry with X-Payment header
```

## Guardrails

- Set `API_PRICE_CENTS` high enough to cover inference cost per request
- Monitor USDC balance — if credits run low, the automaton enters low_compute mode
- Rate-limit by IP to prevent abuse (add to handleRequest)
- Log all payments for audit trail
- Never serve before verifying payment in production deployments
