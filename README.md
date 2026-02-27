# Table of Content

- [Study on Crypto Subscription Landscape](#study-no-crypto-subscription-landscape)
- [Subscription on x402](#subscription-on-x402)
- [Improvement on Paypay](#improvement-on-paypay)

---

## Study On Crypto Subscription Landscape

We mapped out every type of payer wallet and evaluated all possible subscription methods.

![Wallet subscription flow diagram](./wallet-subscription-flow.png)

### CEX Wallets

For CEX wallets (Binance, Coinbase, OKX, Bitget), there is no way to programmatically pull funds. The exchange is the custodian and doesn't expose withdrawal APIs to third parties. The only option is to generate a deposit address and have the payer send funds — either manually each cycle, or using [Binance's Recurring Send](https://www.binance.com/en/support/announcement/detail/b7a09ab510734796a5a35ea6e631a837) feature (Binance only, automates withdrawals on a daily/weekly/monthly schedule).

### EOA Wallets

EOA wallets (MetaMask, Rabby, Trust Wallet, etc.) are where most of the complexity lives. The user holds their own keys, but there's no programmable logic — every transaction requires an explicit signature. We evaluated every possible approach:

**Approve/permit a subscription contract (chosen).** The payer signs a gasless ERC-2612 `permit` (or falls back to `approve`) granting allowance to a SubscriptionManager contract — not the payee directly. The contract stores the payment policy (amount, frequency, timing) and a backend service calls it each cycle to pull the payment. The contract enforces the policy on-chain and rejects any pull that violates it. This is the same pattern [Stripe uses for stablecoin subscriptions](https://stripe.com/blog/introducing-stablecoin-payments-for-subscriptions). It works with every wallet, is gasless for permit-compatible tokens like USDC, and puts an enforceable contract between payer and payee. See the [Subscription Methods Comparison](./subscription-methods-comparison.md) for why we chose this over other approaches.

**Approve/permit the payee directly (deprecated).** This was our previous implementation. The problem: the payer must approve the full subscription amount upfront (e.g. 12 months), and since the allowance goes directly to the payee, the payee can drain it all at any time. No on-chain enforcement of the payment schedule. We moved to the contract approach to fix this.

**Upgrade to smart account + ERC-7715 spend permission (not chosen).** The best UX on paper — prompt the user to upgrade their EOA to a smart account via EIP-7702, then grant an [ERC-7715 spend permission](https://docs.metamask.io/smart-accounts-kit/0.13.0/concepts/erc7715) where the wallet natively displays "10 USDC/month". But this only works on MetaMask. No other wallet supports the upgrade flow or ERC-7715. It's still in Draft status with zero production dapps. Can't be used as a universal solution. See the [Subscription Methods Comparison](./subscription-methods-comparison.md) for a detailed breakdown.

**Generate a new smart wallet + help migrate (considered).** Generate a smart account for the payer and deploy a migration contract to move their funds over in one transaction. Once on a smart wallet, session keys and spend permission modules become available (see [Wallet Subscription Services](./wallet-subscription-services.md) for which providers support what). But this has heavy onboarding friction — the payer must abandon their existing address, and the migration contract is an additional surface to build and audit.

**Manual send (fallback).** The payer simply sends funds each cycle. Works with any wallet but the worst UX — no automation, no enforcement, the payer must remember to pay.

---

## Subscription on x402

We studied [x402](./x402-subscription-and-session-patterns.md), which uses HTTP 402 + [SIWX (Sign-In With X)](./siwx-sign-in-with-x.md) to gate content behind payments. In x402, the user pays once for a time period and gets a session pass — subsequent requests prove wallet ownership without further payment. This is a pay-once-then-browse model, not the monthly pull-based subscription we need.

---

## Improvement on Paypay

We continue to improve [Paypay](./paypay-solution.md), our crypto subscription payment solution. The key change from the previous version: we moved from approving the payee directly to approving a SubscriptionManager contract that enforces the payment policy on-chain. We also added ERC-2612 `permit` support for gasless authorization, falling back to `approve` for tokens that don't support it. We also built a demo integrating with Stripe as Yash suggested, and added a merchant dashboard for viewing payments, managing redirect URLs and webhooks.

**Merchant Dashboard demo**

https://github.com/yikkai95/paypaydoc/raw/main/demo/merchant-dashboard.mp4

**Subscription Flow demo**

https://github.com/yikkai95/paypaydoc/raw/main/demo/subscription-flow.mp4

### Next Steps

Detect the wallet type and capabilities at connect time and use the best available method:

```
CEX wallet
└─ Generate a deposit wallet address for the user
   └─ User transfers full subscription amount (optional)
      └─ Backend pulls monthly from the deposit wallet

EOA wallet
├─ Supports spend permission? (ERC-7715, Coinbase SpendPerm, etc.)
│  └─ Use wallet-native spend permission
├─ Token supports permit? (ERC-2612)
│  └─ permit → SubscriptionManager contract
└─ Fallback
   └─ approve → SubscriptionManager contract
```
