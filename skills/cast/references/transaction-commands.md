# Transaction Commands

Commands for creating, signing, sending, and inspecting Ethereum transactions.

## Sending Transactions

### send

Sign and publish a transaction.

```bash
cast send <TO> [SIG] [ARGS]... [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `--private-key <KEY>` - Private key for signing
- `--from <ADDRESS>` - Sender address (for hardware wallets)
- `--value <VALUE>` - ETH value to send
- `--gas-limit <GAS>` - Gas limit
- `--gas-price <PRICE>` - Gas price (legacy)
- `--priority-gas-price <PRICE>` - Priority fee (EIP-1559)
- `--nonce <NONCE>` - Transaction nonce
- `--legacy` - Use legacy transaction type
- `--json` - Output as JSON
- `--async` - Don't wait for receipt

**Examples:**
```bash
# Send ETH
cast send 0x... --value 1ether --private-key $PRIVATE_KEY --rpc-url https://eth.llamarpc.com

# Call a contract function
cast send 0x... "transfer(address,uint256)" 0xrecipient 1000000 --private-key $PRIVATE_KEY --rpc-url https://eth.llamarpc.com

# Approve ERC20 token
cast send 0xtoken "approve(address,uint256)" 0xspender 1000000000000000000 --private-key $PRIVATE_KEY --rpc-url https://eth.llamarpc.com
```

### mktx

Build and sign a transaction without publishing.

```bash
cast mktx <TO> [SIG] [ARGS]... [OPTIONS]
```

**Options:** Same as `send`

**Example:**
```bash
cast mktx 0x... "transfer(address,uint256)" 0xrecipient 1000000 --private-key $PRIVATE_KEY --rpc-url https://eth.llamarpc.com
# Returns: signed raw transaction hex
```

### publish

Publish a raw transaction to the network.

```bash
cast publish <RAW_TX> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `--async` - Don't wait for receipt

**Example:**
```bash
cast publish 0xf86c... --rpc-url https://eth.llamarpc.com
```

## Reading Transactions

### call

Perform a call on an account without publishing a transaction (read-only).

```bash
cast call <TO> [SIG] [ARGS]... [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `--from <ADDRESS>` - Simulate as this sender
- `--value <VALUE>` - ETH value to simulate
- `-B, --block <BLOCK>` - Block to query
- `--trace` - Show trace output

**Examples:**
```bash
# Get ERC20 balance
cast call 0xtoken "balanceOf(address)(uint256)" 0xowner --rpc-url https://eth.llamarpc.com

# Get token name
cast call 0xtoken "name()(string)" --rpc-url https://eth.llamarpc.com

# Simulate a transfer
cast call 0xtoken "transfer(address,uint256)(bool)" 0xrecipient 1000 --from 0xsender --rpc-url https://eth.llamarpc.com
```

### tx

Get information about a transaction.

```bash
cast tx <TX_HASH> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-f, --field <FIELD>` - Specific field to retrieve
- `-j, --json` - Output as JSON

**Examples:**
```bash
# Get full transaction info
cast tx 0x... --rpc-url https://eth.llamarpc.com

# Get specific field
cast tx 0x... -f gasPrice --rpc-url https://eth.llamarpc.com
```

### receipt

Get the transaction receipt for a transaction.

```bash
cast receipt <TX_HASH> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-f, --field <FIELD>` - Specific field to retrieve
- `-j, --json` - Output as JSON
- `--confirmations <N>` - Wait for N confirmations

**Examples:**
```bash
# Get full receipt
cast receipt 0x... --rpc-url https://eth.llamarpc.com

# Get transaction status (1 = success, 0 = failed)
cast receipt 0x... -f status --rpc-url https://eth.llamarpc.com

# Get gas used
cast receipt 0x... -f gasUsed --rpc-url https://eth.llamarpc.com
```

### decode-transaction

Decode a raw signed EIP-2718 typed transaction.

```bash
cast decode-transaction <RAW_TX>
```

**Example:**
```bash
cast decode-transaction 0x02f86c...
```

## Transaction Utilities

### calldata

ABI-encode a function with arguments (create calldata).

```bash
cast calldata <SIG> [ARGS]...
```

**Examples:**
```bash
cast calldata "transfer(address,uint256)" 0xrecipient 1000000
# Returns: 0xa9059cbb000000000000000000000000...

cast calldata "approve(address,uint256)" 0xspender 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```

### pretty-calldata

Pretty print calldata with decoded arguments.

```bash
cast pretty-calldata <CALLDATA> [OPTIONS]
```

**Example:**
```bash
cast pretty-calldata 0xa9059cbb000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045000000000000000000000000000000000000000000000000000000000001e240
```

### estimate

Estimate the gas cost of a transaction.

```bash
cast estimate <TO> [SIG] [ARGS]... [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `--from <ADDRESS>` - Sender address
- `--value <VALUE>` - ETH value

**Examples:**
```bash
# Estimate gas for transfer
cast estimate 0xtoken "transfer(address,uint256)" 0xrecipient 1000000 --from 0xsender --rpc-url https://eth.llamarpc.com

# Estimate gas for ETH transfer
cast estimate 0xrecipient --value 1ether --from 0xsender --rpc-url https://eth.llamarpc.com
```

### gas-price

Get the current gas price.

```bash
cast gas-price [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
cast gas-price --rpc-url https://eth.llamarpc.com
```

### access-list

Create an access list for a transaction.

```bash
cast access-list <TO> [SIG] [ARGS]... [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `--from <ADDRESS>` - Sender address

**Example:**
```bash
cast access-list 0xtoken "transfer(address,uint256)" 0xrecipient 1000000 --from 0xsender --rpc-url https://eth.llamarpc.com
```

### run

Run a published transaction in a local environment and print the trace.

```bash
cast run <TX_HASH> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-d, --debug` - Open debugger
- `--trace-printer` - Print trace

**Example:**
```bash
cast run 0x... --rpc-url https://eth.llamarpc.com
```

### tx-pool

Inspect the transaction pool (mempool) of a node.

```bash
cast tx-pool [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**
- `status` - Get txpool status
- `inspect` - Inspect pending/queued transactions
- `content` - Get full transaction details

**Example:**
```bash
cast tx-pool status --rpc-url https://eth.llamarpc.com
```

## Common Use Cases

1. **Send tokens** - Use `send` with ERC20 transfer function
2. **Read contract state** - Use `call` for view functions
3. **Debug transactions** - Use `run` to trace execution
4. **Prepare offline signing** - Use `mktx` to create, then `publish` later
5. **Estimate costs** - Use `estimate` and `gas-price` before sending
