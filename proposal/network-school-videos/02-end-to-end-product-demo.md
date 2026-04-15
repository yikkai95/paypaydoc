# End-to-End Product Demo

Last updated: 2026-03-12

| Field | Detail |
|---|---|
| Audience | Yash, mixed audience |
| Length | 1-2 min |
| Goal | Show the whole flow clearly without requiring deep crypto knowledge |
| Main question answered | "What exactly happens from setup to payment?" |

## Must Include

- merchant identity and plan details:
  - Acme
  - `Multisig`
  - 1 USDC per minute
  - one billing cycle in the demo
- merchant settings:
  - payee address
  - Gnosis Safe address
  - approver emails
- checkout flow:
  - wallet already connected
  - subscriber clicks `Subscribe`
  - subscriber signs USDC `permit()`
  - backend handles transaction submission
  - subscriber does not pay gas in this demo
- post-checkout operations:
  - dashboard polls every 10 seconds
  - approvers receive email
  - approver reviews or simulates in Safe
  - approver clicks `Approve`
  - transaction is submitted once Safe threshold is met

## What To Show On Screen

1. Merchant dashboard
2. Settings page
3. Checkout page
4. Dashboard polling or updated status
5. Email notification if visible
6. Safe review / simulation / approval

## What To Say

- clearly separate who is acting:
  - merchant
  - subscriber
  - approver
  - backend service
- explain what is automatic and what is manual
- emphasize the sequence, not protocol internals

## What To Avoid

- overexplaining `permit()`
- discussing unsupported wallet types unless asked
- security deep dive
