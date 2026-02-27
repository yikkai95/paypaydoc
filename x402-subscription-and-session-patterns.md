---
created: 2026-02-26
---

Research from [DeepWiki Q&A on coinbase/x402](https://deepwiki.com/search/is-there-any-subscription-serv_38a4c63b-7030-48f5-af81-0e289e953bd7) exploring how [[x402]] handles subscriptions and session-based access.

## The problem with V1

In [[x402]] V1, every time you access gated content you have to make a wallet transaction and pay. This is a terrible UX — imagine paying a separate onchain tx for every article you read or every API call you make.

## What "subscription" means in x402

**This is not a Stripe-style recurring pull model.** Nobody is pulling money from your wallet every month. Instead, it works like a **membership pass**:

1. You make a one-time payment for a period of time
2. You receive a **session** (a time-limited pass proving you've paid)
3. With that session you can revisit the content freely — no more per-request wallet transactions

The session is proved via [[SIWX — Sign-In With X|Sign-In-With-X (SIWx)]] (based on CAIP-122), which lets you sign a message to prove wallet control without triggering an onchain tx each time. This is the key V2 improvement — turning pay-per-request into pay-once-then-browse.

### Additional patterns

- **Dynamic pricing with subscription tiers** — Headers like `x-subscription: pro/free` can affect pricing, enabling tiered access

