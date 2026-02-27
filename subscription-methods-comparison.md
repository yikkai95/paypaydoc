# Subscription Payment Methods — Comparison Table

Last updated: 2026-02-27

---

## Method 1: USDC `permit()` (ERC-2612)

| | |
|---|---|
| **How it works** | Off-chain EIP-712 signature → gasless allowance to our SubscriptionManager → `transferFrom` each billing cycle |
| **Wallet type** | EOA wallets |
| **Supported wallets** | MetaMask, Rabby, Rainbow, Trust Wallet, Phantom, Zerion, Brave, OKX, Ambire, Ledger (via MetaMask/Rabby), Trezor (via MetaMask/Rabby) |
| **Subscriber action** | Sign 1 off-chain message |
| **Gas cost** | Zero (gasless signature) |
| **Phase** | **Phase 1 — Primary EOA path** |

**Pros:**
- Gasless — subscriber pays zero gas
- Single signature — best possible UX for EOAs
- Works with ~85% of all wallet users
- Battle-tested — every DeFi protocol uses this pattern
- Simplest infrastructure — just RPC + keeper cron
- USDC supports permit natively on all major chains

**Cons:**
- Wallet shows raw EIP-712 data — no subscription terms visible in signing prompt
- Smart wallets cannot use it (USDC's permit lacks EIP-1271 support)
- Allowance is persistent until revoked — if contract compromised, full approved amount at risk (mitigated by scoping to `price × N months`)

**Why we're implementing:** Universal EOA coverage with zero gas. No better alternative exists for EOA wallets.

---

## Method 2: ERC-20 `approve()` (Smart Wallet Path)

| | |
|---|---|
| **How it works** | On-chain `approve()` tx sets allowance to our SubscriptionManager → `transferFrom` each billing cycle |
| **Wallet type** | Smart wallets |
| **Supported wallets** | Safe, Kernel (ZeroDev), Biconomy Nexus, Coinbase Smart Wallet, Ambire, Porto, OKX (post-7702) |
| **Subscriber action** | Approve 1 transaction (gas sponsored by Paymaster) |
| **Gas cost** | Zero for subscriber (Paymaster sponsors ~150K gas) |
| **Phase** | **Phase 1 — Primary smart wallet path** |

**Pros:**
- Universal — every smart wallet on every chain supports ERC-20 approve
- Subscriber pays zero gas (Paymaster)
- Same contract logic as permit path — one SubscriptionManager for both
- Works as fallback for any wallet type

**Cons:**
- Requires on-chain transaction (gas cost for us, though cheap on L2)
- Wallet shows raw approve data — no subscription semantics
- Needs bundler + paymaster infrastructure for gas sponsorship

**Why we're implementing:** Only universal method for smart wallets. Permit doesn't work for smart accounts.

---

## Method 3: ERC-20 `approve()` (EOA Fallback)

| | |
|---|---|
| **How it works** | Same as Method 2 but for EOA wallets when permit is unavailable |
| **Wallet type** | EOA wallets (fallback) |
| **Supported wallets** | All EOA wallets |
| **Subscriber action** | Approve 1 transaction |
| **Gas cost** | ~46K gas (~$0.001 on Base L2) |
| **Phase** | **Phase 1 — Fallback only** |

**Pros:**
- Works when permit fails or token doesn't support ERC-2612
- Universal — every wallet supports it

**Cons:**
- Subscriber pays gas (unless we sponsor via meta-tx relay)
- Worse UX than permit — requires on-chain tx instead of just signing
- Extra complexity to handle gas sponsorship for EOAs

**Why we're implementing (as fallback):** Safety net if permit signature fails for any reason.

---

## Method 4: MetaMask ERC-7715 (`erc20-token-periodic`)

| | |
|---|---|
| **How it works** | `wallet_grantPermissions` RPC call → DeleGator caveat enforcer installed → wallet enforces "X USDC per Y days" natively |
| **Wallet type** | MetaMask (post-7702 smart account upgrade only) |
| **Supported wallets** | MetaMask extension, MetaMask Mobile |
| **Subscriber action** | Approve in wallet UI — sees "10 USDC/month to Acme Corp" |
| **Gas cost** | Zero (Paymaster) |
| **Phase** | **Phase 2** |

**Pros:**
- Best trust UX — wallet natively displays subscription terms in signing prompt
- Subscriber's wallet enforces rules, not our contract
- Production-ready on 10+ mainnets (Base, Polygon, Arbitrum, etc.)
- ~70% browser wallet market share
- Open-source DeleGator framework (MIT + Apache-2.0)

**Cons:**
- MetaMask only — no other wallet supports ERC-7715
- User must upgrade to smart account first (one-time 7702 delegation)
- Uses DeleGator framework, not standard ERC-7579 — separate integration path
- DeleGator caveat enforcer needs custom development and audit
- Not all MetaMask users will have upgraded to smart account

**Why NOT Phase 1:**
- Only benefits MetaMask smart account users (subset of MetaMask users)
- Requires building custom caveat enforcer contract
- approve/permit already works for all MetaMask users
- Adds a second integration path to maintain from day 1
- Phase 1 checkout UI already explains subscription terms clearly — the wallet-native display is a UX improvement, not a requirement

---

## Method 5: Coinbase SpendPermissions

| | |
|---|---|
| **How it works** | SpendPermissionManager singleton added as wallet owner → enforces periodic allowance with params: token, amount, period, start, end |
| **Wallet type** | Coinbase Smart Wallet only |
| **Supported wallets** | Coinbase Wallet (mobile + extension) |
| **Subscriber action** | Approve spend permission — sees periodic spend UI |
| **Gas cost** | Zero (Paymaster) |
| **Phase** | **Phase 2** |

**Pros:**
- Purpose-built for subscriptions — native period-based allowance with automatic reset
- Clean architecture — ERC-712 signatures compatible with EIP-1271
- Passkey authentication (Face ID / fingerprint)
- Growing user base on Base (~10-15% of wallet users)
- Coinbase actively pushing smart wallets as default

**Cons:**
- Coinbase Smart Wallet only — not interoperable with any other wallet
- Proprietary system — not ERC-7715, not ERC-7579
- SpendPermissionManager must be added as wallet owner (trust implication)
- Separate integration to build and maintain
- Sub Accounts feature still maturing

**Why NOT Phase 1:**
- Only benefits Coinbase Smart Wallet users (~10-15%)
- approve() already works for Coinbase Smart Wallet via Paymaster
- Proprietary system means yet another integration path
- Adds complexity without expanding wallet coverage
- Better to ship Phase 1 fast, add Coinbase-specific UX later

---

## Method 6: ERC-7579 Session Keys

| | |
|---|---|
| **How it works** | Install subscription policy module (IActionPolicy) in subscriber's wallet → grant session key with combined policies (CallPolicy + RateLimitPolicy + TimestampPolicy) → our keeper uses session key to call `transfer()` directly |
| **Wallet type** | ERC-7579 smart wallets |
| **Supported wallets** | Safe (via Safe7579), Kernel (ZeroDev), Biconomy Nexus, OKX Wallet (post-7702) |
| **Subscriber action** | Install module + grant session key |
| **Gas cost** | Zero (Paymaster) |
| **Phase** | **Phase 2** |

**Pros:**
- Best trust model — subscriber's own wallet enforces all billing rules
- Cross-wallet standard — works across 4+ wallet brands (Safe, Kernel, Nexus, OKX)
- Most flexible policy system — combine any constraints (amount, rate, time, function, gas)
- No approve/allowance needed — session key calls `transfer()` directly
- Lazy session key enablement — signed off-chain, installed on first use (gasless)
- Rhinestone Module Registry provides on-chain security attestations

**Cons:**
- Small user base — only ~5-10% of users have ERC-7579 smart wallets
- Higher gas per charge — 150-200K gas per UserOp (3x more than transferFrom)
- Requires bundler + paymaster infrastructure
- Module installation has gas cost (one-time)
- Rhinestone Smart Sessions framework still maturing
- Cannot delegate MetaMask/Coinbase/Ledger users to 7579 wallets via 7702

**Why NOT Phase 1:**
- Only ~5-10% of users have compatible smart wallets
- approve() already works for these wallets — session keys are a UX/trust upgrade
- Requires building + auditing a custom ERC-7579 subscription module
- Adds significant infrastructure complexity (bundler, paymaster, session storage)
- Creating a new smart wallet for EOA users doesn't solve it — USDC is in the EOA, not the new wallet
- 3x gas cost per charge compared to simple transferFrom

---

## Method 7: Porto Custom Permissions

| | |
|---|---|
| **How it works** | Porto's custom variant of `wallet_grantPermissions` → session key with spend limits → passkey-signed via WebAuthn/P256 |
| **Wallet type** | Porto wallet (Ithaca/Paradigm) |
| **Supported wallets** | Porto wallet only |
| **Subscriber action** | Passkey biometric prompt |
| **Gas cost** | Zero |
| **Phase** | **Phase 2+** |

**Pros:**
- No browser extension needed — embedded passkey wallet
- Passkey-native authentication (WebAuthn/P256)
- Privacy features (stealth addresses)
- Built on EIP-7702 from the ground up
- Backed by Paradigm

**Cons:**
- Still experimental — "Coming soon" status
- 5th fragmentation vector — incompatible with MetaMask, Coinbase, and ERC-7579 systems
- Unknown user base — brand new wallet
- Custom permission system means yet another integration
- Limited production deployment data

**Why NOT Phase 1 or 2:**
- Still experimental with no production deployment
- Unknown user adoption — no market share
- Adds a 5th incompatible permission system to support
- Risk of building for a platform that may not gain traction
- Will re-evaluate when Porto reaches production and shows adoption numbers

---

## Method 8: Server-side Policy Engine

| | |
|---|---|
| **How it works** | MPC wallet's internal policy engine automates `approve()` + `transfer()` on schedule via server-side rules |
| **Wallet type** | MPC / Institutional wallets |
| **Supported wallets** | Fireblocks, Fordefi, Turnkey |
| **Subscriber action** | Programmatic — no human action needed |
| **Gas cost** | Varies |
| **Phase** | **Supported via approve() in Phase 1** |

**Pros:**
- Enterprise-grade MPC security (HSM-backed)
- Compliance features (AML/KYT, approval workflows)
- Fully automated — no human interaction per charge
- Multi-chain support

**Cons:**
- Institutional product — not consumer-facing
- Different paradigm — server automation, not wallet-native permissions
- High cost (enterprise pricing)
- No standard API — each provider has different policy engine

**Why not a separate integration:** These wallets work fine with our standard approve() path. The BO or their treasury team calls `approve()` via the MPC wallet's API — no special integration needed from our side.

---

## Method 9: Embedded Wallet (Privy / Dynamic)

| | |
|---|---|
| **How it works** | Create passkey wallet for user → wallet backend is configurable (Kernel, Safe, Coinbase) → inherits backend's subscription capabilities |
| **Wallet type** | Embedded (configurable) |
| **Supported wallets** | Privy, Dynamic (with Kernel, Safe, or Coinbase backend) |
| **Subscriber action** | Social login (email/Google/phone) + approve or sign |
| **Gas cost** | Zero (Paymaster) |
| **Phase** | **Phase 2** |

**Pros:**
- Onboards non-crypto users — no existing wallet needed
- Social login (email, Google, Apple, phone)
- Inherits best subscription method from chosen backend
- Privy acquired by Stripe — strong ecosystem alignment
- Solves the "exchange withdrawal only" user problem

**Cons:**
- Creates a new wallet — doesn't help users with existing wallets
- USDC must be deposited into the new embedded wallet first
- Custodial trust — key shares managed by provider (MPC-based)
- Middleware layer — not a subscription system itself
- Extra friction: create wallet → fund wallet → then subscribe

**Why NOT Phase 1:**
- Phase 1 targets users who already have wallets with USDC
- Embedded wallet requires solving the funding problem (deposit from exchange)
- Adds significant integration complexity (Privy/Dynamic SDK)
- Custodial model is a different trust posture than our non-custodial design
- Better to prove the product works for wallet users first, then expand

---

## Summary Matrix

| # | Method | Phase | Wallet Coverage | Why This Phase |
|---|---|---|---|---|
| 1 | USDC `permit()` | **Phase 1** | ~85% (all EOAs) | Only gasless universal EOA method |
| 2 | ERC-20 `approve()` (smart wallet) | **Phase 1** | ~15% (all smart wallets) | Only universal smart wallet method |
| 3 | ERC-20 `approve()` (EOA fallback) | **Phase 1** | 100% (fallback) | Safety net |
| 4 | MetaMask ERC-7715 | Phase 2 | ~70% browser (UX upgrade) | MetaMask-only, needs custom caveat enforcer |
| 5 | Coinbase SpendPermissions | Phase 2 | ~10-15% (UX upgrade) | Coinbase-only, proprietary system |
| 6 | ERC-7579 Session Keys | Phase 2 | ~5-10% (trust upgrade) | Small user base, high infra complexity |
| 7 | Porto Custom Permissions | Phase 2+ | TBD | Still experimental, no production data |
| 8 | Server-side Policy Engine | Via approve() | Institutional | Works with standard approve, no special integration |
| 9 | Embedded Wallet | Phase 2 | Non-crypto users | Needs funding flow, adds complexity |

**Phase 1 achieves 100% wallet coverage with 2 methods (permit + approve).**
**Phase 2 adds wallet-native UX upgrades transparently — same API, same webhooks for BOs.**
