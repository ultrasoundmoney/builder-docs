# Bundle Tracing RPC

Query the lifecycle of a previously submitted bundle by its `bundle_hash`. Trace data typically becomes available a few minutes after the target block.

## ultrasound_getBundleTrace

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "ultrasound_getBundleTrace",
  "params": [
    {
      "bundle_hash": "0x164d7d41f24b7f333af3b4a70b690cf93f636227165ea2b699fbb7eed09c46c7"
    }
  ]
}
```

### Response

```rust
#[derive(serde::Serialize)]
#[serde(rename_all = "camelCase")]
struct GetBundleTraceResponse {
    /// The last fully-indexed block number. Clients distinguish
    /// "doesn't exist" from "not yet indexed" by comparing their target_block
    /// to this value: `data` is null in both cases, but only the latter will
    /// catch up.
    indexed_up_to_block: u64,
    data: Option<BundleTrace>,
}

#[derive(serde::Serialize)]
#[serde(rename_all = "camelCase")]
struct BundleTrace {
    status: BundleStatus,
    /// Coinbase delta attributable to this bundle in the build it was included
    /// in (if any). Decimal-string-encoded U256 wei.
    builder_payment: Option<U256>,
    /// Failure detail when `status` is `Invalid` or `SimulationFail`.
    error: Option<String>,
    /// Hash of the bundle tx whose unauthorized failure caused simulation to
    /// fail. Set only when `status = SimulationFail` AND the failure was
    /// attributable to a specific tx (a non-`revertingTxHashes`,
    /// non-`droppingTxHashes` tx that reverted, halted, or hit another error).
    /// None otherwise.
    failing_tx_hash: Option<B256>,
    /// Parent state the trace was run against, plus whether that parent is
    /// the canonical-chain parent at `target_block`. None for bundles that
    /// didn't reach parent-aware evaluation (e.g. parse failures).
    trace_parent: Option<TraceParent>,
    /// Timestamp from when we received the HTTP headers for the RPC call.
    /// Serialized as an RFC 3339 string.
    received_at: DateTime<Utc>,
    /// Timestamp of the earliest relay submission the bundle was included in.
    /// Serialized as an RFC 3339 string.
    submitted_at: Option<DateTime<Utc>>,
    /// Block number of the build the trace describes. None if the bundle never
    /// reached a build (parse failure or no-trace `Received`).
    block_number: Option<u64>,
    /// Echo of the `blockNumber` parameter from the original `eth_sendBundle`
    /// submission. None if the bundle didn't provide one.
    target_block: Option<u64>,
}

#[derive(serde::Serialize)]
#[serde(rename_all = "camelCase")]
struct TraceParent {
    parent_hash: B256,
    canonical: bool,
}

/// Serialized as PascalCase
#[derive(serde::Serialize)]
enum BundleStatus {
    Received,
    Invalid,
    SimulationFail,
    SimulationPass,
    ExcludedFromBlock,
    IncludedInBlock,
    Submitted,
    Landed,
}
```

Example JSON:

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "indexedUpToBlock": 23040123,
    "data": {
      "status": "Submitted",
      "builderPayment": "12934799399930",
      "error": null,
      "failingTxHash": null,
      "traceParent": {
        "parentHash": "0xabc…",
        "canonical": true
      },
      "receivedAt": "2025-08-01T12:34:48.503639Z",
      "submittedAt": "2025-08-01T12:34:48.603639Z",
      "blockNumber": 23040121,
      "targetBlock": 23040121
    }
  }
}
```

`data` is `null` when the bundle is unknown OR when it has not yet been indexed up to its target block. Clients distinguish the two by comparing their known `targetBlock` to `indexedUpToBlock`.

## Bundle Statuses

```
Received
    |
    +---> Invalid
    |
    +---> SimulationFail
    |
    +---> SimulationPass
    |         |
    |         +---> ExcludedFromBlock
    |         |
    |         +---> IncludedInBlock
    |                   |
    |                   +---> Submitted
    |                             |
    |                             +---> Landed
```

- **Received** — bundle ingested; no build evaluation yet (typically too late in the slot).
- **Invalid** — pre-EVM rejection (parse failure, wrong chain id, stale block, spent nonce).
- **SimulationFail** — top-of-block simulation rejected the bundle. `error` carries the revert reason, halt reason, or other simulator-level cause.
- **SimulationPass** — simulation passed, but the bundle wasn't picked up by any build algorithm before the slot's deadline.
- **ExcludedFromBlock** — evaluated by at least one build, not chosen for any candidate.
- **IncludedInBlock** — included in a candidate build that was not the one submitted to the relay.
- **Submitted** — a block carrying the bundle was sent to a relay.
- **Landed** — the bundle landed on chain.
