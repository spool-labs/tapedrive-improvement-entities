---
time: '004'
title: Segment Rent Paid in TAPE
authors: Spool Labs
category: Storage
type: Core
status: Draft
created: 2025-07-21
---

## Summary

TIME‑004 replaces the lamport‑per‑byte rent with a segment rent paid in TAPE tokens. Every `128‑byte` segment would cost fractional TAPE units per block. Rent is removed from `tape.balance` at the block rate. When the balance reaches zero the tape data is blanked, freeing miners from storing it. 

Importantly, the empty tape account remains so historic tape data can still be recovered through the method defined in [TIME‑003](https://github.com/spool-labs/tapedrive-improvement-entities/blob/main/proposals/TIME-003.md).

## Motivation

The lamport model charged very little and never expired, so the archive grew without limit. Segment rent ties cost to live data, keeps disk use predictable, and gives miners steady income. Users who want longer retention can fund as much TAPE as they wish—even centuries—directly supporting the network.

### Rent Calculation

* Tape rent per block
  `rent = tape.total_segments × RENT_PER_SEGMENT`
* Archive reward per block
  `reward = archive.segments_stored × RENT_PER_SEGMENT`
* Owed since last payment
  `owed = rent × (current_block − tape.last_rent_block)`

### Miner Workflow

* Store segments while `tape.balance > 0`.
* Drop data immediately after a tape is blanked but retain an empty merkle root for future mining challenges on that tape.
* Claim the per‑block reward based on `archive.segments_stored`.

