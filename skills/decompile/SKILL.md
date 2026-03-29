---
name: decompile
description: Decompile EVM bytecode into Solidity source code using the evmdecompiler.com API.
---

# EVM Bytecode Decompiler

Decompile EVM smart contract bytecode into human-readable Solidity source code.

## When to use

Use this skill when the user asks to:
- Decompile EVM bytecode
- Convert bytecode to Solidity
- Reverse-engineer a smart contract from its bytecode or address
- Understand what a contract does given its bytecode or address

## API details

- **Base URL:** `https://decompiler.d23e.ch`
- **API Key:** `eX/wnKCUagiqbBIc9grwDAWWlSRNo9W/YPgh7YFv+RA=`
- **Submit endpoint:** `POST /api/decompile/plugin`
- **Status endpoint:** `GET /api/decompile/full/status/{job_id}`

## Workflow

Follow these steps exactly:

### Step 1: Get the bytecode

**If the user provides raw bytecode:** It should be a hex string starting with `0x`. Use it directly.

**If the user provides a contract address:** Fetch the bytecode using a public RPC endpoint. Use the chain ID to pick the right RPC:

| Chain | Chain ID | RPC URL |
|-------|----------|---------|
| Ethereum | 1 | `https://eth.llamarpc.com` |
| Optimism | 10 | `https://mainnet.optimism.io` |
| BSC | 56 | `https://bsc-dataseed.binance.org` |
| Polygon | 137 | `https://polygon-bor.publicnode.com` |
| Base | 8453 | `https://mainnet.base.org` |
| Arbitrum | 42161 | `https://arb1.arbitrum.io/rpc` |
| Avalanche | 43114 | `https://api.avax.network/ext/bc/C/rpc` |

If the user doesn't specify a chain, assume Ethereum mainnet (chain ID 1).

Fetch the bytecode:

```sh
curl -s -X POST "<RPC_URL>" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getCode","params":["<ADDRESS>","latest"],"id":1}'
```

The bytecode is in the `result` field of the response. If the result is `0x` or empty, the address is not a contract — tell the user.

### Step 2: Submit the bytecode for decompilation (model v6)

Run:

```sh
curl -s -X POST "https://decompiler.d23e.ch/api/decompile/plugin?model=v6" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: eX/wnKCUagiqbBIc9grwDAWWlSRNo9W/YPgh7YFv+RA=" \
  -d '{"bytecode": "<BYTECODE>"}'
```

Replace `<BYTECODE>` with the user's bytecode. The response will be:

```json
{"job_id": "<JOB_ID>"}
```

Save the `job_id`.

### Step 3: Poll for the result

Poll every 5 seconds:

```sh
curl -s "https://decompiler.d23e.ch/api/decompile/full/status/<JOB_ID>" \
  -H "X-API-Key: eX/wnKCUagiqbBIc9grwDAWWlSRNo9W/YPgh7YFv+RA="
```

The response will have a `status` field:

- `"processing"` — still running, keep polling.
- `"done"` — decompilation succeeded. The decompiled Solidity is in `result.result.contract`.
- `"error"` — decompilation failed.

**Timeout:** If the status is still `"processing"` after **30 minutes** (360 polls at 5-second intervals), treat it as a failure for this attempt.

### Step 4: Fallback to default model

If step 3 resulted in an error or timed out, retry with the **default model** by omitting the `model` parameter:

```sh
curl -s -X POST "https://decompiler.d23e.ch/api/decompile/plugin" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: eX/wnKCUagiqbBIc9grwDAWWlSRNo9W/YPgh7YFv+RA=" \
  -d '{"bytecode": "<BYTECODE>"}'
```

Then poll for the result the same way as step 3 (every 5 seconds, 30-minute timeout).

### Step 5: Handle final result

- **If decompilation succeeded** (either attempt): Present the decompiled Solidity code from `result.result.contract` to the user in a Solidity code block. If `result.result.functions` is non-empty, also list the detected function names.
- **If both attempts failed**: Tell the user that decompilation was not possible for this bytecode. Suggest they try at [evmdecompiler.com](https://www.evmdecompiler.com/) directly.

## Important notes

- Always use the exact API key shown above. Do not ask the user for an API key.
- Do not modify or truncate the bytecode before sending it.
- When polling, use exactly 5-second intervals. Do not poll more frequently.
- The bytecode can be very large. Always pass it in the JSON body, never as a URL parameter.
