---
time: '002'
title: Updatable Tapes and Finalization Semantics
authors: Spool Labs
category: Storage
type: Core
status: Dev-complete
created: 2025-07-11
---

## Summary

This TIME introduces **updatable tapes**, a new tape type that allows data owners to make changes to tape segments prior to finalization. Finalized tapes remain immutable and are eligible for inclusion in the TAPEDRIVE archive and mining reward schedule. This mechanism supports dynamic data use cases such as compressed token accounts while preserving the archive's integrity guarantees.

## Motivation

The original TAPEDRIVE design enforced tape immutability at write time. While simple, this rigid model limited expressiveness for use cases that benefit from staged data construction, such as compressed account trees and multi-phase initialization processes.

This proposal allows tape writers to build data incrementally and then **finalize it** once complete, without sacrificing the permanence and auditability guarantees of the archive.

## Detailed Design

### Tape States

Tapes now exist in one of two states:

**Updatable:**  
The tape is mutable by its owner. Segments can be added, overwritten, or removed within defined constraints.

**Finalized:**  
The tape is sealed. No further writes are allowed. It becomes eligible for inclusion in the canonical archive and for miner validation and reward calculations.

Only **finalized tapes** are considered part of the TAPEDRIVE archive and included in Merkle root indexing.

### Finalization Rules

- Finalization is irreversible
- Finalization may be triggered by the tape creator at any time
- Once finalized, a tape’s contents are treated as immutable and may be referenced in proofs and archival indexing structures

### Miner Considerations

Because a tape may be finalized at any time, miners are expected to **track updatable tapes alongside finalized ones**

This ensures the following:

- New finalizations are not missed
- Tapes become eligible for validation as soon as they are finalized
- The archive state remains consistent across the network

### Use Cases

Updatable tapes enable support for the following:

- Compressed token account mechanics
- Batching or staged data writes
- Protocols that require append-only logs prior to sealing

Without this feature, such patterns would require complex client-side workarounds or duplicate archive writes.

## Impact

This proposal has the following impacts:

- Introduces a new tape lifecycle state and finalization API
- Requires miners to expand indexing to include updatable tapes
- Adds complexity to tape creation logic, but no impact on finalized data integrity

