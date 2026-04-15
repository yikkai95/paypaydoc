# Risks / Constraints / Open Questions

Last updated: 2026-03-12

| Field | Detail |
|---|---|
| Audience | Balaji + Swaym |
| Length | 1-2 min |
| Goal | Show that the team understands trade-offs and knows what still needs validation |
| Main question answered | "What still needs to be decided before production?" |

## Must Include

- operational trade-offs:
  - Safe approval adds latency
  - recurring billing may not scale if every cycle requires manual approval
- security trade-offs:
  - bounded allowance vs open-ended allowance
  - payee-direct approval vs contract-enforced approval
- wallet-coverage limits:
  - USDC `permit()` works well for EOAs
  - not a universal path for smart-account subscribers
- UX limits:
  - wallet signature prompt still shows raw signing data
  - the merchant workflow is more complex than a pure autopay model
- explicit decision questions:
  - should Safe approve every cycle?
  - what is the production spender model?
  - what revoke path should exist for subscribers?

## What To Show On Screen

1. One summary slide
2. Optional quick return to Safe / dashboard to illustrate the operational step

## What To Avoid

- pretending these questions are already solved
- turning this into a pure marketing video
