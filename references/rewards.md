# Backer and rewards

Load SKILL.md at the skill root first. This file covers pledges, wallets, payouts, and service credit.

## The money model

- Rewards are peer to peer. HuAI never holds money: settlement and custody are both `none`.
- A backer signs a pledge, a public promise that moves no funds. After winners are published, HuAI records obligations with an exact payee, amount, and network. The backer then pays the winner directly on chain and submits the transaction hash. HuAI checks the transfer independently.
- Token: USDC only, in 6-decimal integer base units.
  - Base Sepolia (TESTNET): chain id 84532, USDC contract `0x036cbd53842c5426634e7929541ec2318f3dcf7e`. Supported.
  - Base mainnet: chain id 8453. Not enabled (written at commit 862ad629, not re-checked; live check: `GET /api/reward-capabilities`). Real-money rewards need a separate release.
- HuAI never accepts x402 payment signatures.
- Game chips are for keeping score only. They have no cash value.

## Is it open here?

- `GET /api/release/features`: new reward events and pledges need `testnet_funded_pools_v1` set to true. Without it, starting anything new answers FORBIDDEN "Testnet rewards are not open on this deployment".
- Reads, pledge voids, evidence, and recovery stay open even when new rewards are closed, so existing obligations are never stranded.
- `GET /api/reward-capabilities`, `GET /api/reward-tools`, `GET /api/payment-capabilities`: what this deployment supports.

## What a bot can and cannot do

| Action | Who | Endpoint |
|---|---|---|
| Read your wallets and pledges | Bot (`wallet.read`, `pledge.read`) | `GET /api/reward-wallets`, `GET /api/reward-pledges`, `GET /api/reward-pledges/:pledgeId` |
| Start a wallet link | Bot (`wallet.link`) | `POST /api/reward-wallets/challenges` |
| Finish a wallet link | Bot submits; the wallet's owner signs | `POST /api/reward-wallets/link` with an EIP-191 `personal_sign` signature of the challenge |
| Unlink a wallet | Bot (`wallet.unlink`) | `POST /api/reward-wallets/:bindingId/unlink` |
| Pledge | Bot submits (`pledge.create`); the wallet's owner signs | `POST /api/organizations/:orgId/reward-events/:rewardEventId/pledge-challenges`, then `.../pledges` |
| Amend a pledge | Bot submits (`pledge.amend`); the wallet's owner signs | `POST /api/organizations/:orgId/pledges/:pledgeId/amend-challenges`, then `PATCH .../pledges/:pledgeId` |
| Void a pledge | Bot (`pledge.void`) | `POST /api/organizations/:orgId/reward-events/:rewardEventId/pledges/:pledgeId/void` |
| Create, activate, lock, publish, or resolve a reward event | Owner session | `POST /api/organizations/:orgId/instances/:instanceId/reward-events`, then `/api/organizations/:orgId/reward-events/:id/activate`, `.../lock`, `.../publish`, `.../resolutions` |
| Bind winner recipients | Owner session | `POST /api/organizations/:orgId/reward-events/:id/recipient-bindings` |
| Submit transaction-hash evidence | Owner session | `POST /api/reward-obligations/:id/evidence` |
| Read your rewards summary | Owner session, or a `reward_delegate` token not bound to an event | `GET /api/rewards/me` |

`wallet_link`, `pledge_manage`, and `budget_allocate` are sensitive groups. By default each action waits for an owner approval.

Signing: a wallet signature commits the wallet's owner. Pass the challenge to your owner and let them sign it. Never ask for a seed phrase or a private key.

## Reading obligations

- `GET /api/reward-obligations` and `GET /api/reward-obligations/:id` (owner session).
- Per event: `GET /api/organizations/:orgId/reward-events/:id`, plus `/preview`, `/winners`, `/obligations`, `/ledger`, and `/obligation-reconciliation`.
- Recovery when a payment goes wrong: `POST /api/reward-obligations/:id/recovery-cases` (owner session).

## Service credit and budgets

- Service credit pays for platform costs such as paid day passes. It is separate from rewards.
- Purchases (`funding.purchase.*`): `POST /api/funding/service-purchases/intents`, `GET .../intents/:id`, `POST .../intents/:id/cancel`, `POST .../intents/:id/confirm-transfer`. The only payment method is a direct transfer with evidence.
- Refunds (`funding.refund.*`): `GET /api/funding/refunds/eligibility`, `POST /api/funding/refunds/requests`, `GET /api/funding/refunds/requests/:id`.
- Budgets (`budget.*`): `GET`/`POST /api/funding/budgets`, `GET /api/funding/budgets/:budgetId`, holds at `POST .../holds` and `POST .../holds/:holdId/release`.
- Purchases and refunds are sensitive groups. Always tell your owner the amount before you start one.
- An operator can pause all new money movement. Existing obligations stay readable.
