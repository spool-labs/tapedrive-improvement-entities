---
timd: '003'
title: Slot Number Linked List for Tape Write Ordering
authors: Spool Labs
category: Storage
type: Core
status: Draft
created: 2025-07-15
---

## Summary

This TIMD introduces a **slot number linked list** structure to improve tape traversal and verification in TAPEDRIVE. It proposes capturing the **slot number** of each tape write or update and including it in Merkle tree leaves, along with storing the **first write slot number** in the tape account metadata. This replaces the CLI-level transaction signature chaining with a more compact, protocol-enforced solution.

## Motivation

The current TAPEDRIVE CLI uses a linked-list approach based on transaction signatures. Each tape write includes the previous transaction signature, which allows backward traversal without special indexing infrastructure.

However, this is not enforced at the program level. Transaction signatures are also relatively large at 64 bytes and cannot be referenced inside Solana programs because the SVM does not support transaction introspection. While SIMD-0258 proposes to introduce introspection, it is not yet available.

Using slot numbers instead of transaction signatures offers several benefits. Slot numbers are small at 8 bytes, can be accessed through the Clock sysvar, and can be enforced on-chain. This change provides a standard method for miners to traverse tapes backwards while reducing data overhead.

## Detailed Design

### Slot Number Linked List

Every tape write or update instruction will record the previous slot number (the slot in which the prior write occurred) and store it inside the Merkle tree leaf. The current slot number will be obtained from the Clock sysvar. Additionally, the first write slot number will be saved in the tape account metadata.

This forms a program-enforced linked list using slot numbers. It allows tapes to be reconstructed in reverse order by following the slot history.

### Traversal and Verification Flow

Miners can traverse any tape backwards by starting from the latest write slot, which is stored in the tape account. From there, miners can query the Solana RPC for the block associated with that slot, parse transactions in that block, and find the transaction that wrote to the tape account. The miner can then follow the recorded previous slot number field to continue traversing backwards until reaching the first write slot number, which serves as a natural stopping point.

Even if multiple writes per account per block were to be introduced in Solana in the future, the traversal remains feasible because only one transaction can write to an account per block today.

### Merkle Leaf Format Update

Each Merkle tree leaf will now include:

- Tape segment data
- Previous slot number (8 bytes)
- Current slot number (8 bytes)

The total additional storage per leaf is 16 bytes.

### Tape Metadata Update

Tape accounts will have a new field:

- `first_slot: u64` which records the slot number of the first write

This gives miners an upper and lower bound on the blocks they need to query when reconstructing a tape.

## Alternatives Considered

- **Transaction signature chaining** as used in the current CLI provides backwards traversal but has no program enforcement and high storage costs.
- **Waiting for SIMD-0258** would require new Solana features and would not be available for some time.

## Impact

This change improves tape traversal, enforces consistent write history at the program level, and reduces the storage overhead compared to transaction signature chaining. It creates a predictable way for miners to reconstruct tapes without relying on off-chain infrastructure.

## Security Considerations

Slot numbers are deterministic and verifiable through standard Solana RPC calls. They cannot be manipulated by clients. Using slots instead of transaction signatures eliminates reliance on ephemeral or non-program-accessible identifiers.

If Solana introduces multiple writes to an account within a single block in the future, miners may need to parse full blocks to identify writes, but the current one-write-per-account rule ensures practical feasibility today.

