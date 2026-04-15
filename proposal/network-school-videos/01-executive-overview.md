# Executive Overview

Last updated: 2026-03-12

| Field | Detail |
|---|---|
| Audience | Balaji |
| Length | 30-45s |
| Goal | Show what the solution is, why it matters, and why it is safer than naive approval-to-payee flows |
| Main question answered | "Why should I care?" |

## Must Include

- one-line problem statement:
  - recurring crypto subscriptions are difficult because wallets do not support trusted periodic pull payments well
- one-line solution statement:
  - subscriber signs once with USDC `permit()`, backend handles submission, and merchant-side pull is controlled by Safe
- simple happy path:
  - merchant configures plan
  - subscriber clicks `Subscribe`
  - subscriber signs once
  - Safe approver approves
  - merchant receives funds
- why this is useful:
  - better UX than manual payment every cycle
  - better control than approving the payee directly
  - gasless for the subscriber in this demo
- one high-level comparison frame:
  - manual payment each cycle is universal but poor UX
  - direct approve / permit to payee is simple but unsafe
  - wallet-native periodic permissions are elegant but fragmented across wallets
  - our current flow is the practical middle ground that works today
- one clear limitation:
  - this version still depends on a merchant-side approval workflow

## High-Level Comparison To Include

Use one simple comparison slide before the product screens.

| Option | Strength | Weakness | Executive takeaway |
|---|---|---|---|
| Manual payment every cycle | Works with any wallet | Bad UX, no automation | Too much user friction |
| Approve / permit payee directly | Simple implementation | Merchant can drain the allowance | Too much trust in merchant |
| Wallet-native periodic permissions | Best trust UX | Only works on specific wallet ecosystems | Not universal enough today |
| Our current flow | Works now, better control, gasless subscriber path in this demo | Still has merchant-side approval overhead | Best practical option today |

## Suggested Comparison Message

The executive message should be:

- there is no perfect universal crypto subscription primitive today
- the choice is between usability, safety, and wallet coverage
- our current flow is not the final end state, but it is the best pragmatic version we can ship now

## What To Show On Screen

1. One comparison slide with the four options above
2. Merchant dashboard with the `Multisig` plan visible
3. Checkout page with the connected wallet
4. Wallet signing step
5. Safe approval page
6. Final dashboard state or success state

## What To Say

- focus on outcome, not implementation
- use plain words like `sign once`, `approve`, `receive funds`
- keep protocol references minimal
- position our flow as the pragmatic option available today, not as the perfect long-term answer

## What To Avoid

- EIP-heavy explanation
- too many competitor names or protocol names
- contract architecture
- nonce, replay, spender, calldata detail
