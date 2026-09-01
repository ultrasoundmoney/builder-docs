# Bundle RPC

JSON-RPC endpoints for submitting orderflow to Ultra Sound Builder.

## Rate limits

Unauthenticated requests are limited to 50 requests per minute per source IP. We are prioritizing onboarding additional orderflow: [contact us](README.md#questions) for a free API token and much higher limits, then include it in the `X-Api-Key` header.

## Bundle inclusion

Cielago considers candidate bundles greedily in descending order of their simulated total payment to the builder's coinbase address, after any configured refund. It does not rank by payment per unit of gas; lower gas use only breaks ties. Bundles are included when they fit the block's gas, blob-gas, nonce, and execution constraints.

## eth_sendBundle

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendBundle",
  "params": [
    {
      txs,                // Array[String], signed transactions (hex) to execute atomically; may be empty to cancel a bundle
      blockNumber,        // (Optional) String, hex-encoded target block number; defaults to next block
      revertingTxHashes,  // (Optional) Array[String] or null, tx hashes allowed to revert or be discarded
      droppingTxHashes,   // (Optional) Array[String] or null, tx hashes allowed to be discarded but not revert
      replacementUuid,    // (Optional) String, identifier for replacing or canceling this bundle; `uuid` is accepted as an alias
      refundPercent,      // (Optional) Number, 0–99; percent of refund-tx ETH reward to refund
      refundRecipient,    // (Optional) Address, refund destination; defaults to first tx sender
      refundTxHashes      // (Optional) Array[String] or null, max 1; the tx whose coinbase delta is the refund basis. Defaults to the last tx
    }
  ]
}
```

`revertingTxHashes`, `droppingTxHashes`, and `refundTxHashes` may be omitted, set to `null`, or provided as arrays. Omitted and `null` values are treated as empty arrays.

A bundle with an empty `txs` array and a `replacementUuid` cancels the bundle carrying that identifier. The response is a bundle hash computed over the empty transaction set.

Response:

```jsonc
{
  "result": {
    "bundleHash": "0x164d7d41f24b7f333af3b4a70b690cf93f636227165ea2b699fbb7eed09c46c7"
  },
  "error": null,
  "id": 1
}
```

### Bundle hash

Builder-computed identifier for the bundle. `revertingTxHashes` and `droppingTxHashes` are treated as sets — their listing order does not affect the hash.

## eth_cancelBundle

Cancels a previously submitted bundle by its `replacementUuid`.

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_cancelBundle",
  "params": [
    {
      replacementUuid  // String, the UUID provided when the bundle was submitted; `uuid` is accepted as an alias
    }
  ]
}
```

Response:

```jsonc
{ "result": 200, "error": null, "id": 1 }
```

## eth_sendRawTransaction

Submit a signed raw transaction for private inclusion.

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendRawTransaction",
  "params": [
    "0x…"  // String, signed raw transaction (hex)
  ]
}
```

Response:

```jsonc
{ "result": 200, "error": null, "id": 1 }
```

## eth_chainId

Returns the chain ID the builder is configured for.

```jsonc
{ "jsonrpc": "2.0", "id": 1, "method": "eth_chainId", "params": [] }
```

Response:

```jsonc
{ "result": "0x1", "error": null, "id": 1 }
```
