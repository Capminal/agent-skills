---
name: Vhermes-Opus-Funder
description: "Autonomous yield-to-inference bridge: harvest onchain LP fees or staking rewards, route earnings into Venice Opus AI credits with cost gating, spend tracking, and mode switching."
version: 1.0.0
author: retarddegeneth
tags: [vhermes, autonomous, yield, inference, venice, fee-claiming, base]
---

# Vhermes Opus Funder

An autonomous agent that turns onchain yield into offchain AI inference.

## Purpose

Most agent skills stop at "swap tokens" or "provide liquidity." This one closes the loop:

yield (LP fees / staking rewards) -> claim -> route -> inference credits -> reasoning output

It maintains a sustainable computation budget without requiring manual funding. Every cent of yield is tracked against every cent of compute spent.

## Core Loop

Every tick:

1. **Detect yield** — read FeeLocker `availableFees` or staking contract claimable balance via read-only RPC call.
2. **Claim** — if claimable >= `CLAIM_THRESHOLD`, submit claim transaction and wait confirmation.
3. **Route** — decide based on `daily_fee_rate`:

   | daily_fee_rate | Mode | Action |
   |---------------|------|--------|
   | < LOW_THRESHOLD | idle | skip, log only |
   | >= LOW_THRESHOLD | accumulate | LP all yield into target pool |
   | >= HIGH_THRESHOLD | build | stake yield for inference credits |

4. **Cost gate** — before any spend exceeding `MAX_SINGLE_COST_DIEM`:
   - Check current sVVV balance via `stakedBalanceOf`
   - If sVVV >= threshold: fast path (free llama) allowed
   - Else: estimate cost in DIEM, confirm remaining yield covers it
5. **Inference call** — call Venice API with generated bearer, log cost to `memory/opustx.jsonl`.
6. **Drift check** — if identity or config drift exceeds 0.70 Jaccard, halt and alert operator.

## Required Contracts (Base Mainnet)

| Contract | Address | Purpose |
|----------|---------|---------|
| NFPM v3 | 0x03a520b32C04BF3bEEf7BEb72E919cf822Ed34f1 | LP position management |
| FeeLocker | 0xF7d3BE3FC0de76fA5550C29A8F6fa53667B876FF | Unclaimed LP fees |
| DIEM | 0xF4d97F2da56e8c3098f3a8D538DB630A2606a024 | Yield token / inference payment |
| Venice VVV Staking | defined by env `VVV_STAKING_ADDRESS` | Staking for inference credits |

## Selectors

- FeeLocker `availableFees`: `0x8296535a`
  - Calldata: `abi.encode(feeOwner, token)`

## Environment Variables

| Variable | Required | Purpose |
|----------|----------|---------|
| `RPC_URL` | yes | Base mainnet RPC endpoint |
| `PRIVY_APP_ID` | yes | Privy server wallet app ID |
| `PRIVY_APP_SECRET` | yes | Privy server wallet secret |
| `PRIVY_WALLET_ID` | yes | Privy wallet to operate |
| `VVV_STAKING_ADDRESS` | yes | Venice VVV staking contract |
| `DIEM_TOKEN_ADDRESS` | optional | Override DIEM address |
| `VENICE_API_KEY` | optional | Skip autonomous bearer mint |
| `AGENT_PRIVATE_KEY` | local testing | Bypass Privy for local runs |

## Mode Thresholds

Set per agent in config:

```yaml
claim_threshold_diem: "1.0"       # min DIEM claimable
low_mode_threshold_diem_per_day: "0.5"
high_mode_threshold_diem_per_day: "2.0"
max_single_cost_diem: "0.05"
```

## Cost Log Schema

Log every inference call to `memory/opustx.jsonl`:

```json
{"ts":"2026-06-24T00:00:00Z","provider":"venice","model":"claude-opus-4-7","cache_hit":false,"latency_ms":1200,"tokens_in":500,"tokens_out":200,"cost_diem":0.027,"yield_at_call":42.5,"mode":"build"}
```

## Safety Constraints

- Read-only checks first; only submit transactions after yield >= claim threshold AND expected cost is covered.
- All signing goes through Privy server wallet; no raw private keys.
- If `opustx.jsonl` shows cumulative cost_diem > cumulative yield_diem for 24h, halt and alert operator.
- Identity files under `identity/` are genesis-locked; halt on Jaccard drift > 0.30.

## Commands

```bash
# Tick: detect yield, claim, route, infer
npm run opus:tick

# Dry run: same as tick but skip all writes
npm run opus:tick -- --dry

# Read-only portfolio + yield summary
node --env-file=.env --import tsx scripts/check-portfolio.ts

# Earnings + inference cost report
node --env-file=.env --import tsx scripts/opustx-report.ts
```

## Integration with Capminal Skills

This skill assumes wallet and contract-interaction are available. It does NOT re-implement:
- Token transfers (use `capminal` skill)
- Generic contract calls (use `contract-interaction` skill)
- Gas estimation (delegate to Capminal's underlying RPC tools)

It adds the autonomous decision layer: when to claim, how to route, when to spend.
