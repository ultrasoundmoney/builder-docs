# Bundle RPC

JSON-RPC endpoints for submitting orderflow to Ultra Sound Builder.

Adding a valid auth token in the `X-Api-Key` header increases your rate limit. 

## eth_sendBundle

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendBundle",
  "params": [
    {
      txs,                // Array[String], signed transactions (hex) to execute atomically
      blockNumber,        // (Optional) String, hex-encoded target block number; defaults to next block
      revertingTxHashes,  // (Optional) Array[String], tx hashes allowed to revert or be discarded
      droppingTxHashes,   // (Optional) Array[String], tx hashes allowed to be discarded but not revert
      replacementUuid,    // (Optional) String, identifier for replacing or canceling this bundle
      refundPercent,      // (Optional) Number, 0–99; percent of refund-tx ETH reward to refund
      refundRecipient,    // (Optional) Address, refund destination; defaults to first tx sender
      refundTxHashes      // (Optional) Array[String], max 1; the tx whose coinbase delta is the refund basis. Defaults to the last tx
    }
  ]
}
```

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
      replacementUuid  // String, the UUID provided when the bundle was submitted
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
