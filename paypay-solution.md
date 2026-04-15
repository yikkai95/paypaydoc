# Paypay — Our Solution

Paypay is our crypto subscription payment solution built on top of the research in this repo.

## Demo Videos

**Merchant Dashboard** — merchant page for viewing dashboard, managing redirects and webhooks

https://github.com/yikkai95/paypaydoc/raw/main/demo/merchant-dashboard.mp4

**Subscription Flow** — end-to-end subscription payment demo integrated with Stripe

https://github.com/yikkai95/paypaydoc/raw/main/demo/subscription-flow.mp4

## Network School Follow-up

We also documented the first `USDC permit() + Gnosis Safe multisig` Loom requested for the Network School follow-up.

- Short Loom (27s): https://www.loom.com/share/1c6a68f0302b4c329288c93558b225b9
- Long Loom (1m 19s): https://www.loom.com/share/b0f7e850308f42cdabe43b3b3ae35801
- Meeting note: [meetings/2026-03-12.md](./meetings/2026-03-12.md)

## What We Built

### Merchant Dashboard

A merchant-facing page where business owners can:

- View subscription payment dashboard
- Manage redirect URLs (post-payment redirects)
- Configure webhooks for payment event notifications

### Stripe Integration

Following Yash's suggestion, we built a demo integrating with Stripe's checkout flow. This allows merchants to use Stripe as the frontend while our contract handles the on-chain subscription logic.

### Subscription Contract (SubscriptionManager)

The payer grants token allowance to a SubscriptionManager contract — not the payee directly. The contract stores the payment policy (amount, frequency, timing) and enforces it on-chain. A backend service calls the contract each billing cycle to pull the payment. The contract rejects any pull that violates the policy.

**Authorization flow:**
1. Try ERC-2612 `permit` — gasless off-chain signature (works for USDC, DAI, etc.)
2. Fall back to ERC-20 `approve` — on-chain transaction if permit is not supported

## What Changed From the Previous Version

The previous version used **approve/permit directly to the payee** + backend `transferFrom`. The problem: the payer had to approve the full subscription amount upfront (e.g. 12 months), and the payee could drain the entire allowance at any time.

We changed to **approve/permit to a contract** + backend triggers the contract. The contract sits between payer and payee and enforces the payment schedule on-chain. The payee can only withdraw through the contract, which rejects pulls at the wrong timing or for the wrong amount.
