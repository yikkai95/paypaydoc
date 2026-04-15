# Subscriber-Facing Demo

Last updated: 2026-03-12

| Field | Detail |
|---|---|
| Audience | Actual subscriber |
| Length | 30-60s |
| Goal | Build trust, show the action required, and reduce fear at signing time |
| Main question answered | "What am I agreeing to, and what do I need to do?" |

## Must Include

- what the subscriber is buying:
  - plan name
  - price
  - billing frequency or billing period shown in the demo
- what the subscriber needs to do:
  - connect wallet
  - click `Subscribe`
  - sign once
- what happens next:
  - service processes the subscription
  - merchant approval happens in the background
- trust points:
  - no gas paid by the subscriber in this demo
  - the subscriber is not manually sending a payment every cycle
  - approval is reviewed by the merchant-side workflow before execution
- if implemented:
  - how to stop or revoke the subscription later

## What To Show On Screen

1. Checkout page
2. Wallet signature prompt
3. Short confirmation state

Only briefly show merchant-side screens if needed for context.

## What To Say

- use non-technical language
- explain that the subscriber is authorizing a subscription flow, not manually paying every cycle
- say exactly what the subscriber sees and clicks

## What To Avoid

- Safe internals
- Gnosis terminology unless necessary
- contract jargon
- anything that sounds scary or overly technical

## Important Caveat

Do not say the subscriber can cancel, revoke, or edit the subscription later unless that path is actually available and tested.
