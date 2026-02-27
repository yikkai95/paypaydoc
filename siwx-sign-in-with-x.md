---
created: 2026-02-26
categories: ["[[Researches]]"]
---

# SIWX — Sign-In With X

SIWX is a chain-agnostic authentication primitive. The user signs a challenge message with their wallet, the server verifies the signature, and now the server knows which address it's talking to. That's the entire idea.

The "X" means it works across chains — [[Ethereum]], [[Solana]], [[Bitcoin]], whatever supports message signing. One protocol, multiple ecosystems.

## What It Is

A standardized HTTP handshake for proving wallet ownership.

1. Server sends a challenge (nonce + domain + timestamp).
2. User signs the challenge in their wallet (MetaMask, Phantom, etc.).
3. Server verifies the signature against the claimed address.
4. Server issues a session — cookies, JWTs, whatever you use.

No passwords. No [[OAuth]] provider. No email verification. The wallet is the identity.

## What It Is Not

SIWX is authentication only. It does not:

- Give your server access to the user's private key.
- Allow your server to sign transactions on behalf of the user.
- Authorize spending, token transfers, or approvals.
- Interact with the blockchain at all (it's off-chain signature verification).

A SIWX session proves "this HTTP client controls wallet 0xABC." Nothing more.

## How It Differs From SIWE

[[SIWE]] (Sign-In With Ethereum, [[EIP-4361]]) is Ethereum-specific. SIWX generalizes the pattern to any chain. The message format and verification logic adapt to whatever signature scheme the chain uses (secp256k1 for EVM, ed25519 for [[Solana]], etc.), but the HTTP-level flow stays the same.

## Use Cases

**Authentication without passwords.** Replace email/password with wallet signatures. No credential database to leak. Identity is a key pair the user already controls.

**Gating access to resources.** Combine with on-chain state: user paid USDC, holds an NFT, or is a DAO member → server checks wallet → grants access. API endpoints, content, downloads, dashboards.

**Pay-once access passes.** User makes a single on-chain payment. Server records the payment with an expiration (e.g., 30 days). Subsequent requests include a SIWX proof — server checks the signature and the stored expiration. No re-payment until the pass expires. This is the [[x402]] V2 pattern.

**Cross-chain identity.** One user session that aggregates wallets across [[Ethereum]], [[Solana]], [[Bitcoin]]. Prove ownership of multiple addresses under a single auth context.

**Linking on-chain identity to off-chain systems.** "This Discord user is 0xABC." "This API caller holds governance token Y." Bridges web2 services to on-chain state without exposing keys.

## Example Flow (x402 Time-Limited Access)

```
1. Client requests protected resource.
2. Server responds: 402 Payment Required.
3. Client pays once (EIP-3009 or SPL transfer).
4. Server records: wallet 0xABC paid, valid until <now + 30 days>.
5. Client sends SIWX proof in SIGN-IN-WITH-X header on subsequent requests.
6. Server verifies signature, checks stored expiration, grants access.
7. After 30 days, session expires. Client must pay again.
```

SIWX handles steps 5-6. The payment (step 3) and expiration logic (step 4, 7) are separate concerns.

## What SIWX Does Not Solve

**Recurring payments.** SIWX cannot pull funds from a wallet. Blockchains are push-based — the user must sign every transfer. For auto-billing you need one of:

- `approve()` + `transferFrom()` (ERC-20 allowance pattern)
- [[ERC-4337]] session keys with spending caps
- Superfluid token streams
- Off-chain payment intents with smart wallet policies

**On-chain authorization.** Proving you own an address and authorizing that address to spend are orthogonal operations. SIWX handles the first. The second requires separate on-chain transactions signed by the user's wallet.

## Summary

SIWX is one thing: proof of wallet ownership over HTTP. It replaces the entire traditional auth stack (passwords, [[OAuth]], email verification) with a single cryptographic primitive. Everything else — access control, payment verification, session expiration — is application logic you build on top.
