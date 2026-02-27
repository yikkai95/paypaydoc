# Wallet Subscription Services — Who Provides What

Last updated: 2026-02-27

This table maps each wallet type to the services/providers that enable subscription payments for it. Only subscription-relevant capabilities are listed.

---

## Service Providers Legend

| Service | What It Does For Subscriptions |
|---|---|
| **Rhinestone** | ERC-7579 modules: Smart Sessions (session keys with spending limit + timeframe + usage limit policies), Scheduled Transfers executor, Module Registry for security attestations |
| **ZeroDev** | Kernel smart account with session key policies: CallPolicy, RateLimitPolicy, TimestampPolicy, GasPolicy. Combine for subscription semantics |
| **Pimlico** | Bundler (relays subscription UserOps to chain) + Paymaster (sponsors gas for subscription txs). Infrastructure layer — no subscription logic itself |
| **Biconomy** | Nexus smart account + Smart Sessions (co-developed with Rhinestone) + MEE for cross-chain execution. Also supports EOA via EIP-7702 Fusion |
| **Coinbase CDP** | SpendPermissionManager contract: periodic allowance with auto-reset (token, allowance, period, start, end, spender) |
| **MetaMask Delegation Toolkit** | Caveat enforcers: `erc20PeriodTransfer` (periodic spend cap), `erc20Streaming` (linear unlock), `allowedTargets`, `allowedMethods`, `limitedCalls` |
| **Loop Crypto** | Complete subscription processor: authorize → auto-charge → dunning → settlement. Integrates with Stripe/Chargebee |
| **BoomFi** | Complete subscription gateway: smart contract authorization → auto-charge → notifications → fiat/crypto settlement |
| **Superfluid** | Continuous per-second streaming via Super Tokens (wrapped ERC-20). Different model — not periodic charges |
| **Tempo** | L1 blockchain optimized for payments. Not a service — a chain where subscription contracts can be deployed with priority blockspace |

---

## EOA Browser Wallets

| Wallet | Available Subscription Services | Subscription Method | Notes |
|---|---|---|---|
| **MetaMask (EOA mode)** | Loop Crypto, BoomFi | `permit()` or `approve()` → service pulls via `transferFrom` | No session keys in EOA mode. Loop/BoomFi handle the full billing cycle |
| **MetaMask (smart account via 7702)** | MetaMask Delegation Toolkit, Loop Crypto, BoomFi | `erc20PeriodTransfer` caveat enforcer via ERC-7715, or standard `approve()` | Delegation Toolkit enables wallet-native "10 USDC/month" display. Loop/BoomFi work via standard approve path |
| **Rabby** | Loop Crypto, BoomFi | `permit()` or `approve()` only | No AA, no session keys, no 7702 upgrade. Only standard approve/permit services work |
| **Rainbow** | Loop Crypto, BoomFi | `permit()` or `approve()` only | Same as Rabby — pure EOA, no AA features |
| **Trust Wallet (EOA mode)** | Loop Crypto, BoomFi | `permit()` or `approve()` only | Despite 7702 support, no subscription framework documented |
| **Trust Wallet (7702 mode)** | Pimlico (bundler/paymaster) | `approve()` via UserOp | Custom AA implementation — not ERC-7579, no Rhinestone/ZeroDev compatibility |
| **Phantom (EVM)** | Loop Crypto, BoomFi | `permit()` or `approve()` only | No AA on EVM side. Solana has native session keys but irrelevant for EVM subscriptions |
| **Zerion** | Loop Crypto, BoomFi | `permit()` or `approve()` only | Pure EOA — no AA features |
| **Brave** | Loop Crypto, BoomFi | `permit()` or `approve()` only | Pure EOA — no AA features |
| **OKX Wallet (EOA mode)** | Loop Crypto, BoomFi | `permit()` or `approve()` only | Can fall back to standard path |
| **OKX Wallet (7702 mode)** | Rhinestone, ZeroDev, Pimlico, Biconomy | ERC-7579 session keys with subscription policies | OKX co-authored ERC-7579. Full module compatibility. Best EOA-to-smart-account story |

---

## Smart Wallets (Native Smart Accounts)

| Wallet | Available Subscription Services | Subscription Method | Notes |
|---|---|---|---|
| **Safe** | Rhinestone, ZeroDev, Pimlico, Biconomy, Loop Crypto, BoomFi | ERC-7579 Smart Sessions (via Safe7579 adapter), or standard `approve()` | Rhinestone Smart Sessions = session key with spending limit + timeframe policies. $100B+ secured. Most battle-tested |
| **Kernel (ZeroDev)** | ZeroDev, Rhinestone, Pimlico, Biconomy, Loop Crypto, BoomFi | Session keys: CallPolicy + RateLimitPolicy + TimestampPolicy, or standard `approve()` | Most advanced policy system. 6M+ accounts. Lazy session key enablement (gasless install) |
| **Biconomy Nexus** | Biconomy, Rhinestone, Pimlico, Loop Crypto, BoomFi | Smart Sessions + MEE execution, or standard `approve()` | Co-developed Smart Sessions with Rhinestone. MEE enables cross-chain subscription execution |
| **Coinbase Smart Wallet** | Coinbase CDP, Loop Crypto, BoomFi | SpendPermissionManager (native periodic allowance), or standard `approve()` | Purpose-built subscription contract. NOT ERC-7579 — Rhinestone/ZeroDev modules don't work here |
| **Ambire** | Pimlico, Loop Crypto, BoomFi | Session keys (custom implementation), or standard `approve()` | First wallet with 7702 from day 1. Custom AA — not confirmed ERC-7579 compatible |

---

## Hardware Wallets

| Wallet | Available Subscription Services | Subscription Method | Notes |
|---|---|---|---|
| **Ledger (via MetaMask/Rabby)** | Loop Crypto, BoomFi | `permit()` or `approve()` — must confirm on physical device | **Session keys fundamentally incompatible** — every tx needs device confirmation. 7702 restricted to EF contract only |
| **Trezor (via MetaMask/Rabby)** | Loop Crypto, BoomFi | `permit()` or `approve()` — must confirm on physical device | Same as Ledger. 7702 support in progress but will restrict delegation targets |

---

## Passkey / Embedded Wallets

| Wallet | Available Subscription Services | Subscription Method | Notes |
|---|---|---|---|
| **Porto (Ithaca)** | None standard — custom system | Custom `wallet_grantPermissions` with session keys | Experimental. Own permission system — incompatible with Rhinestone, ZeroDev, MetaMask, Coinbase |
| **Privy embedded** | Rhinestone, ZeroDev, Pimlico, Biconomy, Coinbase CDP (depends on backend) | Inherits from configured smart account backend | Privy is middleware. If backend = Kernel → ZeroDev session keys. If backend = Coinbase → SpendPermissions |
| **Dynamic embedded** | Rhinestone, ZeroDev, Pimlico, Biconomy (depends on backend) | Inherits from configured smart account backend | Same as Privy — capabilities depend on chosen smart account |
| **Turnkey** | None native — infrastructure layer | `approve()` via API signing | Key management infra only. Subscription logic must be built on top with a smart account framework |

---

## MPC / Institutional Wallets

| Wallet | Available Subscription Services | Subscription Method | Notes |
|---|---|---|---|
| **Fireblocks** | Internal policy engine | `approve()` automated via server-side policies | Enterprise-only. Policies automate recurring charges — not wallet-native permissions |
| **Fordefi** | Internal policy engine | `approve()` automated via server-side policies | Same as Fireblocks — institutional, server-side automation |

---

## Service Compatibility Matrix

Which services work with which wallets:

| Service | MetaMask EOA | MetaMask Smart | Rabby | Rainbow | OKX Smart | Safe | Kernel | Nexus | Coinbase SW | Ledger | Porto | Privy/Dynamic |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Rhinestone** | — | — | — | — | Yes | Yes | Yes | Yes | — | — | — | Via backend |
| **ZeroDev** | — | — | — | — | Yes | Yes | Yes | Yes | — | — | — | Via backend |
| **Pimlico** | — | — | — | — | Yes | Yes | Yes | Yes | — | — | — | Via backend |
| **Biconomy** | — | Via Fusion | — | — | Yes | Yes | Yes | Yes | — | — | — | Via backend |
| **Coinbase CDP** | — | — | — | — | — | — | — | — | Yes | — | — | Via backend |
| **MetaMask Delegation** | — | Yes | — | — | — | — | — | — | — | — | — | — |
| **Loop Crypto** | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | — | — |
| **BoomFi** | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | — | — |
| **Superfluid** | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | — | Yes |
| **Tempo** | — | — | — | — | Yes | Yes | Yes | — | — | — | — | — |

---

## Key Takeaways

1. **Loop Crypto and BoomFi** are the only complete subscription services that work across all wallet types — they use standard `approve()`/`permit()` under the hood
2. **Rhinestone + ZeroDev + Pimlico** form the ERC-7579 subscription stack — but only for smart wallets (Safe, Kernel, Nexus, OKX)
3. **MetaMask Delegation Toolkit** is MetaMask-only — powerful (`erc20PeriodTransfer` is built for subscriptions) but siloed
4. **Coinbase CDP SpendPermissions** is Coinbase-only — purpose-built for subscriptions but not interoperable
5. **Most EOA wallets (Rabby, Rainbow, Phantom, Zerion, Brave)** have zero subscription-specific service support beyond standard approve/permit
6. **Hardware wallets** can only work with services that use standard approve — session keys are fundamentally incompatible with physical signing
7. **Tempo** is a chain, not a service — it provides priority blockspace for payment transactions but you still need subscription logic on top
8. **Our service fills the gap**: a complete subscription solution (like Loop/BoomFi) with the best integration DX (like Stripe) — payment links, script tags, middleware, SDK
