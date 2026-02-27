# Paypay — Our Solution

Paypay is our crypto subscription payment solution built on top of the research in this repo.

## Demo Videos

- [Merchant Dashboard](./demo/merchant-dashboard.mp4) — merchant page for viewing dashboard, managing redirects and webhooks
- [Subscription Flow](./demo/subscription-flow.mp4) — end-to-end subscription payment demo integrated with Stripe

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
