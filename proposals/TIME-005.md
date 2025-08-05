---
time: '005'
title: Packx Proof-of-Access Algorithm
authors: Spool Labs
category: Mining
type: Core
status: Draft
created: 2025-08-05
---

## Summary

TIME-005 introduces the "packx" proof-of-access algorithm to ensure miners store and commit to tape data encoded uniquely against their public key. This prevents miners from using public RPC endpoints to mine against shared data and mitigates Sybil attacks. The algorithm enforces unique data commitments via a new merkle root stored in a miner's spool account, making it computationally and storage-efficient to store packx-encoded segments rather than raw data.

## Motivation

The packx algorithm addresses the risk of miners exploiting shared data sources, ensuring fairness and security in the protocol. By requiring miners to commit to uniquely encoded tape data before mining, TIME-005 strengthens the integrity of the mining process and prevents Sybil attacks, fostering a balanced and decentralized network.


### Miner Workflow

* Encode tape data using packx with the miner’s public key to generate unique segments.
* Commit to a new merkle root in the spool account for the encoded tape data.
* Store packx-encoded segments to efficiently respond to mining challenges.
* Submit solutions to prove access to the encoded data.
* Verify solutions on-chain by reconstructing the data and checking the solution’s hash difficulty.

The packx algorithm ensures miners maintain unique, verifiable data commitments, enhancing the protocol’s security and fairness.
