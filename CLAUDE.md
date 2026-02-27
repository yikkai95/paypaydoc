# Crypto Subscription Payments Research

## About This Repo

This is a research documentation repo, not a codebase. All content is written in Markdown targeting a developer audience.

## Writing Guidelines

- Keep language concise and technical — the reader is a developer.
- Use short paragraphs. Prefer bullet points and tables over long prose.
- When describing a project or solution, always cover: how it works, pros, cons, and limitations.
- Use code snippets or pseudocode where it helps illustrate a mechanism (e.g. tx flow, contract interaction).
- Avoid marketing language. Be objective — state trade-offs clearly.

## Structure

- `README.md` — main research document: problem statement, wallet types, subscription methods, existing projects, comparison matrix.
- `wallet-subscription-services.md` — detailed breakdown of which wallet providers support subscriptions and which services (Rhinestone, ZeroDev, Coinbase CDP, Loop Crypto, BoomFi, etc.) work with which wallets.
- `subscription-methods-comparison.md` — detailed comparison of each subscription method (permit, approve, ERC-7715, Coinbase SpendPermissions, ERC-7579 session keys, Porto, embedded wallets, etc.) with pros/cons, phased rollout plan, and summary matrix.
- `x402-subscription-and-session-patterns.md` — how x402 handles subscriptions via pay-once + SIWX sessions (not a pull model).
- `siwx-sign-in-with-x.md` — chain-agnostic wallet authentication primitive (CAIP-122) used by x402 for session-based access.
- `paypay-solution.md` — our Paypay solution: merchant dashboard, Stripe integration, contract architecture, what changed from previous version.
- `demo/merchant-dashboard.mov` — demo video of the merchant dashboard.
- `demo/subscription-flow.mov` — demo video of the end-to-end subscription flow.
- `paypay.excalidraw` — visual diagram of wallet types and subscription flow.
- `wallet-subscription-flow.png` — exported PNG of the diagram for README rendering.

## Key Concepts

- **Wallet types under evaluation:** CEX, EOA, Smart Account (ERC-4337), EIP-7702
- **Projects being compared:** Stripe crypto, x402, and others (Superfluid, Sablier, etc.)
- The core question: how can on-chain wallets support recurring "pull" payments without requiring user action each cycle?
