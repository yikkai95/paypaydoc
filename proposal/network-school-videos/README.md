# Network School Videos

Last updated: 2026-03-12

This folder splits the Network School video plan into separate files so each audience-specific version can be reviewed independently.

## Files

- [01-executive-overview.md](./01-executive-overview.md) — Balaji / decision-maker version
- [02-end-to-end-product-demo.md](./02-end-to-end-product-demo.md) — Yash / mixed audience product walkthrough
- [03-technical-flow-architecture.md](./03-technical-flow-architecture.md) — Swaym / engineering walkthrough
- [04-subscriber-facing-demo.md](./04-subscriber-facing-demo.md) — subscriber-facing trust and UX version
- [05-risks-constraints-open-questions.md](./05-risks-constraints-open-questions.md) — trade-offs and unresolved decisions
- [06-failure-edge-case-demo.md](./06-failure-edge-case-demo.md) — optional failure-mode walkthrough

## Shared Guidance

Every version should state:

- recurring crypto pull payments are hard because wallets do not natively support trusted periodic pull payments well
- this demo uses subscriber-side USDC `permit()` plus merchant-side Gnosis Safe approval
- the current flow is a demo, not yet the final production design

Every version should avoid:

- claiming the flow is fully automated unless that is confirmed
- claiming universal wallet support
- promising cancel / revoke / dispute behavior unless already implemented

## Stakeholder Mapping

| Stakeholder | Most Useful Video(s) | Why |
|---|---|---|
| Balaji | Executive overview, Risks / constraints | Needs business value and confidence in the decision |
| Swaym | Technical flow, Failure / edge cases | Needs implementation detail and operational clarity |
| Yash | End-to-end product demo | Needs a clear product walkthrough without deep crypto detail |
| Subscriber | Subscriber-facing demo | Needs trust, clarity, and simple next steps |

## Suggested First Three

If only three videos can be prepared first:

1. Executive overview
2. Technical flow / architecture
3. Subscriber-facing demo

These three cover decision-maker, engineer, and end-user needs.
