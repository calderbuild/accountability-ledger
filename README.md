# accountability-ledger

Public evidence store for [SafeReceipt](https://github.com/calderbuild/SafeReceipt)'s
agent accountability layer, built for BUIDL_QUESTS 2026.

## What this is

`ActionRegistry.sol` (deployed on
[Monad Testnet](https://testnet.monadscan.com) and
[Base Sepolia](https://sepolia.basescan.org)) only stores a hash of each
agent action's outcome on-chain. This repo hosts the full pre-image --
the trace JSON that hashes to that value -- so anyone can independently
re-fetch it, recompute the hash, and compare it to the on-chain record
without trusting SafeReceipt's own frontend.

Each file under `traces/` is one receipt's evidence. The file's git commit
history is itself a timestamp: a trace published *before* a corresponding
on-chain `VERIFIED`/`MISMATCH` transaction is stronger evidence than one
published after.

## What this proves, and what it doesn't

**Proves:**
- The declared intent for an action was committed (hashed on-chain) *before*
  the action ran.
- The published trace has not been altered since it was committed --
  changing a single byte changes the hash, which no longer matches the
  on-chain record.

**Does not prove:**
- That the trace is a complete or honest account of what the agent actually
  did. Nothing here cryptographically stops the harness from mis-reporting
  its own trace -- that would require a TEE or a re-execution proof, neither
  of which is built yet. This is an explicit, disclosed limitation, not an
  oversight: see `SafeReceipt`'s README for the v1 scope boundary.

## Format

```
traces/{receiptId}.json
```

Each file is a `CanonicalDigest`-shaped JSON trace: declared intent,
`PipelineEvent[]` execution log, computed `outcomeHash`, and the
`agentId` (from `AgentIdentityRegistry.sol`) that produced it.

## Verifying a receipt independently

1. Read the on-chain receipt from `ActionRegistry.sol` (`getReceipt(receiptId)`)
   to get `outcomeHash` and `evidenceURI`.
2. Fetch the corresponding `traces/{receiptId}.json` file from this repo.
3. Recompute the hash of the fetched JSON using the same canonicalization
   rules as `SafeReceipt/frontend/src/lib/canonicalize.ts`.
4. Compare it to the on-chain `outcomeHash`. Match = the published record is
   the one that was actually committed on-chain.

The SafeReceipt frontend's "Verify Independently" button does exactly this,
client-side, in front of the viewer.
