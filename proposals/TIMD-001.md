---
timd: '001'
title: Mining Rewards and Block-Based Participation
authors: Spool Labs
category: Economics
type: Core
status: In-Progress
created: 2025-07-11
---

## Summary

This TIMD introduces a redesign of the TAPEDRIVE mining and reward system. It replaces the existing fixed emission model with a write-fee-backed reward stream and introduces block-based mining with consistency-based participation scoring. The new model directly links archived data to long-term miner rewards and adjusts dynamically based on real-time network conditions.

## Motivation

TAPEDRIVE’s initial devnet launch used a static emission curve to distribute rewards to miners, with hardcoded write fees (approximately 1 lamport per byte) that were not tied to rewards. While this allowed early bootstrapping, it lacked economic alignment between storage usage and miner compensation.

This TIMD proposes a new architecture that:

- Ties rewards directly to archived data via write fees
- Moves from open-ended mining to discrete, timed blocks
- Incentivizes consistent participation over burst mining
- Adapts participation and difficulty targets over time

## Detailed Design

### Emission Curve Changes

Under the new model, the fixed emission curve used as a base and will decay over time. When data is written to TAPEDRIVE, a one-time fee is paid to cover 100 years of archival storage. This fee is not distributed immediately but is instead streamed out over time, becoming the primary source of miner rewards.

Each block unlocks one minute of accrued write fees, paid to miners who contributed valid proofs during that block.

This mechanism ensures rewards are backed by real storage demand, creating a long-term, usage-aligned incentive structure without relying on inflation.

### Block-Based Mining

Mining is restructured into discrete blocks, each targeting a duration of approximately one minute. A block is considered complete when a set number of valid proofs, called `target_participants`, have been submitted.

- Miners may submit one proof per block
- If a block is delayed, previously participating miners may submit again to help complete it
- The reward for a block is split evenly among the contributing miners, weighted by consistency

This approach ensures predictable mining intervals, reduces idle time, and simplifies reward scheduling.

### Consistency-Based Rewards

To discourage opportunistic participation and reward reliability, each miner tracks a consistency streak, which is the number of consecutive blocks for which they have submitted a valid proof.

While exact parameters are subject to tuning, the reward scale is linear:

- 32 consecutive submissions results in 100 percent share
- 16 submissions results in 50 percent share
- 1 submission results in approximately 3.1 percent share

If a miner misses blocks, their streak is reduced by one per block missed, resetting to zero if inactive long enough.

Consistency weighting ensures that miners who run persistent, reliable infrastructure are favored over short-term or burst miners.

### Epoch Adjustments

To ensure the system stays responsive, two key parameters are adjusted every epoch:

#### 1. Target Participants (`target_participants`)

- If all submissions in an epoch are from unique miners, increase the target by one
- If any miners appear more than once in a block, decrease the target by one

This helps scale participation targets with actual network capacity.

#### 2. Proof Difficulty

- If blocks are being solved too quickly, increase difficulty
- If blocks are slower than the target, reduce difficulty

Together, these adjustments form a feedback loop that maintains network stability and responsiveness.

## Rationale

This design introduces several key benefits:

- Rewards are usage-backed rather than emission-based
- Storage write fees are put to productive use
- Mining rewards scale naturally with archive growth
- The system favors consistency and uptime instead of raw hashpower
- Network parameters are self-tuning, enabling adaptive load balancing

This aligns TAPEDRIVE’s incentives with its core mission of permanent, decentralized storage.

