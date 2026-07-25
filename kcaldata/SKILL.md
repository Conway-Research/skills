---
name: kcaldata
description: Look up calories and nutrition for any food from a natural-language description. Free tier needs no key; paid calls settle in USDC over x402 on Base.
---

# kcaldata — calorie and nutrition lookup

Turn a plain-language food description into structured, sourced nutrition data.
Send text the way a person would write it — `2 large eggs`, `8 oz salmon`,
`1 cup cooked rice`, `3 slices bacon` — and get back the matched food, calories
per 100 g, the total for the amount given, the grams used, and a plain-language
explanation of how the total was calculated.

Values are derived from the U.S. Department of Agriculture FoodData Central
(SR Legacy) dataset, which is in the public domain. kcaldata is independent and
not affiliated with the USDA.

## When to use this

- You need calorie or nutrition figures for a food, meal, or ingredient.
- You are building or operating anything that logs meals, plans diets, costs out
  recipes, or answers food questions.
- You need a citable source for a nutrition number rather than a guess.

## Free access (no key, no wallet)

```
GET https://api.kcaldata.com/v1/lookup?query=2%20large%20eggs
```

50 calls per day, no signup. Returns JSON:

```json
{
  "query": "2 large eggs",
  "matched_food": "Egg, whole, raw, fresh",
  "calories_kcal_per_100g": 143,
  "calories_kcal": 143,
  "quantity": 2,
  "unit": "large",
  "grams_used": 100.0,
  "basis": "2 × (1 large ≈ 50 g)",
  "confidence": 0.99,
  "source": "USDA FoodData Central (SR Legacy)",
  "alternatives": [{"food": "Egg, white, raw, fresh", "calories_kcal_per_100g": 52}]
}
```

## Paid access via x402 (USDC on Base)

Beyond the free allowance, calls cost **$0.005 USDC** on Base mainnet
(`eip155:8453`). Two ways to pay:

**Paid REST endpoint** — returns HTTP 402 with payment requirements, then the
data once payment settles:

```
GET https://api.kcaldata.com/v1/pro/lookup?query=2%20large%20eggs
```

**MCP tool** — connect the server below and call `lookup_calories`. After the
free allowance the tool returns an x402 payment-required response containing the
price and recipient; attach payment under the `x402/payment` metadata key to
continue.

```
https://api.kcaldata.com/mcp-server/mcp
```

Payment is gasless for the payer (EIP-3009), so a wallet holding only USDC can
pay — no ETH required.

## Notes

- Outputs are estimates for informational purposes only, and are not medical,
  dietary, or nutritional advice.
- Matching is automated and may return an approximate food. The response always
  includes `confidence` and `alternatives` so a caller can sanity-check the match.
- Developer plan (monthly API key, raised limits): 25 USDC/month, contact
  kcaldata@protonmail.com.
- Docs: https://api.kcaldata.com/docs · Terms: https://kcaldata.com/terms
