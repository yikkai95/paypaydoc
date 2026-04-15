# Technical Flow / Architecture

Last updated: 2026-03-12

| Field | Detail |
|---|---|
| Audience | Swaym |
| Length | 2-4 min |
| Goal | Explain how the flow works under the hood and where the main constraints are |
| Main question answered | "How is this implemented, and what assumptions does it rely on?" |

## Must Include

- the actors:
  - subscriber wallet
  - checkout frontend
  - backend service
  - payee
  - Gnosis Safe approvers
  - contract or executor path
- authorization model:
  - what the USDC `permit()` signature authorizes
  - who the spender is
  - whether the spender is the payee directly or a contract
  - whether allowance is bounded by amount / duration
- transaction lifecycle:
  - signature collected at checkout
  - backend creates pending action
  - service updates dashboard
  - service notifies Safe approvers
  - Safe review / simulation / approval
  - execution on-chain
- state split:
  - what is stored or tracked off-chain
  - what is enforced on-chain
- production questions:
  - is Safe approval required every cycle or only at setup / exceptions?
  - how are permit expiry and nonce handled?
  - what is the replay-protection model?
  - what happens if subscriber balance is insufficient?
  - what happens if allowance is exhausted?

## What To Show On Screen

1. One sequence diagram or simple architecture slide
2. Checkout screen
3. Backend-created pending action if visible
4. Dashboard status
5. Safe approval screen

## Suggested Sequence Diagram

```text
subscriber -> frontend: subscribe
frontend -> wallet: sign USDC permit
frontend -> backend: send permit payload
backend -> service db: create pending pull
backend -> merchant dashboard: update status
backend -> approvers: send email
approver -> Safe: review / simulate / approve
Safe -> chain: execute
chain -> payee: transfer USDC
```

## What To Say

- be explicit about what is confirmed and what is still open
- call out any place where the current demo differs from the intended production architecture
- use exact words like `spender`, `allowance`, `execution`, `threshold`

## What To Avoid

- vague phrases like `the backend handles everything`
- hiding uncertainty on spender model or approval frequency
