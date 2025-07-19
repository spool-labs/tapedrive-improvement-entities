---
time: '003'
title: Slot Number Linked List for Tape Write Ordering
authors: Spool Labs
category: Storage
type: Core
status: Draft
created: 2025-07-15
---

## Summary

This TIME introduces a **slot number linked list** to improve how segments within a tape are connected in TAPEDRIVE. Each tape write records the **slot number** of the previous write in its Merkle tree leaf, with the **first write slot number** stored in tape account metadata. This replaces CLI-level transaction signature chaining with a more compact, program-enforced structure. The result is simpler and more reliable tape traversal without relying on large data fields or off-chain tooling.

## Motivation

The current TAPEDRIVE CLI uses a linked list based on transaction signatures to allow reverse traversal of segments. This method is not enforced by the program and uses large 64-byte signatures that cannot be read within the Solana Virtual Machine. 

Slot numbers offer a cleaner alternative. They are compact at 8 bytes, accessible on-chain via the Clock sysvar, and allow for protocol-level enforcement. This proposal improves consistency while reducing storage overhead and dependency on client behavior.

## Detailed Design

### Slot Number Linked List

Every tape write or update will record the **previous slot number** (the slot in which the prior write occurred) inside the Merkle tree leaf. The **current slot number** is read from the Clock sysvar and also included. The **first slot number** is recorded in the tape metadata.

This creates a program-enforced linked list within the tape, enabling backward traversal of segments by following the slot chain.

### Traversal and Verification Flow

Miners reconstruct tapes by starting from the latest write slot recorded in the tape account. They query Solana RPC for the block at that slot, identify the transaction writing to the tape, and follow the previous slot number backwards until reaching the first write slot.

This approach avoids reliance on transaction signatures and remains practical even if Solana introduces multi-writes per block. The one-write-per-account rule today ensures only one transaction modifies a tape account within a slot.

### Merkle Leaf Format Update

Each Merkle tree leaf will include:
- Tape segment data
- Previous slot number (8 bytes)
- Current slot number (8 bytes)

The additional storage overhead is 16 bytes per leaf.

### Tape Metadata Update

Tape accounts will include a new field:
- `first_slot: u64`, marking the slot number of the first write to the tape.

This allows miners to efficiently identify tape boundaries during reconstruction.

## Alternatives Considered

- **Transaction signature chaining** is client-dependent, inefficient, and not enforceable on-chain.
- **Waiting for SIMD-0258 (transaction introspection)** would require new SVM features and delay standardization of tape traversal.

