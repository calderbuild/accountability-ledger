# accountability-ledger

Public evidence store for [SafeReceipt](https://github.com/calderbuild/SafeReceipt)'s
agent accountability layer, built for BUIDL_QUESTS 2026.

## What this is

`ActionRegistry.sol` on [Monad Testnet](https://testnet.monadscan.com) only
stores a hash of each agent action's outcome on-chain. This repo hosts the full pre-image --
the trace JSON that hashes to that value -- so anyone can independently
re-fetch it, recompute the hash, and compare it to the on-chain record
without trusting SafeReceipt's own frontend.

Each file under `traces/` is one receipt's evidence. The SafeReceipt wrapper
pushes the trace and waits until it is readable before it links the outcome
on-chain, so every V2.1 evidence URI resolved at the moment it was written.
Traces are public and unredacted.

## What this proves, and what it doesn't

**Proves:**

- The declared intent for an action was committed (hashed on-chain) _before_
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
traces/v2.1/{receiptId}.json   current ActionRegistry (V2.1)
traces/{receiptId}.json        V2.0 ActionRegistry, kept for its historical receipts
agents/{name}.json             agent metadata (the tokenURI of each agent NFT)
```

Receipt ids restart at 1 on each registry deploy, hence one directory per
version. Each trace holds `receiptId`, `agentId`, `declaredIntent`, the
`events` log (`{stage, message, progress, data, timestamp}`), `durationMs`, and
the `policy` result. The outcome hash is not stored in the file.

## Verifying a receipt independently

1. Read the on-chain receipt from `ActionRegistry.sol` (`getReceipt(receiptId)`)
   to get `outcomeHash` and `evidenceURI`.
2. Fetch the corresponding `traces/{receiptId}.json` file from this repo.
3. Recompute the hash: drop top-level `status`, `linkedTxHash` and
   `outcomeHash` if present, sort keys recursively, serialize as compact JSON,
   keccak256 (`SafeReceipt/frontend/src/lib/v2.ts` `hashTrace`).
4. Compare it to the on-chain `outcomeHash`. Match = the published record is
   the one that was actually committed on-chain.

The SafeReceipt Fleet page's "Verify independently" button does this in the
browser, and also checks that the trace names the same receipt and agent,
re-runs the policy rules, and checks the filer still owns the agent.

## License

MIT
