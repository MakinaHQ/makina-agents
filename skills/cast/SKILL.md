---
name: cast
description: Foundry Cast CLI reference for EVM interactions. Use when encoding/decoding ABI data, querying blockchain state (balances, blocks, transactions), sending transactions, ENS lookups, data conversions (hex, wei, bytes32), or wallet management.
allowed-tools: Bash, Read, Grep, Glob
---

# Cast Skill

Cast is a Swiss Army knife for interacting with Ethereum applications from the command line. It is part of the Foundry toolkit.

## Overview

Cast provides commands for:
- Encoding/decoding ABI data
- Querying blockchain state (accounts, blocks, transactions)
- Sending transactions
- ENS lookups
- Data conversions (hex, wei, bytes32, etc.)
- Contract interaction and inspection
- Wallet management

## Command Categories

Cast commands are organized into the following categories. Each category has a dedicated reference file for detailed command information:

| Category | Reference | Description |
|----------|-----------|-------------|
| ABI | `references/abi-commands.md` | Encode/decode ABI data |
| 4byte | `references/4byte-commands.md` | Signature lookups via openchain.xyz |
| Account | `references/account-commands.md` | Balances, nonces, proxy info |
| Block | `references/block-commands.md` | Block information |
| Chain | `references/chain-commands.md` | Chain ID, client version |
| Code | `references/code-commands.md` | Bytecode inspection |
| Conversion | `references/conversion-commands.md` | hex, wei, bytes32, RLP, UTF8 |
| ENS | `references/ens-commands.md` | ENS lookups |
| Transaction | `references/transaction-commands.md` | Send, call, receipts |
| Crypto | `references/crypto-commands.md` | Keccak, EIP-191 hashing |
| Utility | `references/utility-commands.md` | Storage, CREATE2, selectors |
| Wallet | `references/wallet-commands.md` | Wallet management |

## Common Options

Most Cast commands support:
- `--rpc-url <URL>` - RPC endpoint
- `-e, --etherscan-api-key <KEY>` - Etherscan API key
- `--chain <CHAIN>` - Chain name or ID
- `-h, --help` - Help

## Quick Examples

```bash
# Query balance
cast balance 0x... --rpc-url https://eth.llamarpc.com

# Call contract
cast call 0x... "balanceOf(address)" 0x... --rpc-url https://eth.llamarpc.com

# Send transaction
cast send 0x... "transfer(address,uint256)" 0x... 1000000 --private-key $PK

# Encode calldata
cast calldata "transfer(address,uint256)" 0x... 1000000

# Convert wei to ether
cast from-wei 1000000000000000000

# Get function selector
cast sig "transfer(address,uint256)"
```

## Installation

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

## Documentation

Full docs: https://getfoundry.sh/cast/reference/cast

## Troubleshooting

### Cast crashes with "Attempted to create a NULL object" on macOS

If `cast call`, `cast send`, or other network commands crash with:
```
The application panicked (crashed).
Message:  Attempted to create a NULL object.
Location: .../system-configuration-0.6.1/src/dynamic_store.rs:154
```

**Cause:** Sandboxed environments (like Claude Code's sandbox) block access to macOS System Configuration framework. Foundry's HTTP client tries to read system proxy settings and crashes when this access is denied.

**Solutions:**

1. **Disable sandbox for network calls** - Run cast network commands with sandbox disabled

2. **Use curl with JSON-RPC** - Direct RPC calls work in sandbox:
   ```bash
   # Get function selector (works in sandbox)
   SELECTOR=$(cast sig "balanceOf(address)")

   # Encode the argument
   DATA="${SELECTOR}000000000000000000000000<address-without-0x>"

   # Make the call via curl
   curl -s -X POST https://eth.llamarpc.com \
     -H "Content-Type: application/json" \
     --data "{\"jsonrpc\":\"2.0\",\"method\":\"eth_call\",\"params\":[{\"to\":\"0x...\",\"data\":\"$DATA\"},\"latest\"],\"id\":1}"
   ```

3. **Offline commands work fine** - These don't need network access:
   - `cast sig` - Get function selector
   - `cast calldata` / `cast abi-encode` - Encode calldata
   - `cast abi-decode` / `cast calldata-decode` - Decode data
   - `cast from-wei` / `cast to-wei` - Unit conversions
   - `cast keccak` - Hash data
