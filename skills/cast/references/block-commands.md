# Block Commands

Commands for querying block information from the blockchain.

## Commands

### block

Get information about a block.

```bash
cast block [BLOCK] [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-f, --field <FIELD>` - Specific field to retrieve
- `--full` - Include full transaction objects
- `-j, --json` - Output as JSON

**Examples:**
```bash
# Get latest block
cast block --rpc-url https://eth.llamarpc.com

# Get specific block
cast block 18000000 --rpc-url https://eth.llamarpc.com

# Get specific field
cast block 18000000 -f timestamp --rpc-url https://eth.llamarpc.com

# Get block as JSON
cast block 18000000 -j --rpc-url https://eth.llamarpc.com
```

### block-number

Get the latest block number.

```bash
cast block-number [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
cast block-number --rpc-url https://eth.llamarpc.com
```

### age

Get the timestamp of a block.

```bash
cast age [BLOCK] [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
cast age 18000000 --rpc-url https://eth.llamarpc.com
```

### base-fee

Get the base fee of a block.

```bash
cast base-fee [BLOCK] [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
cast base-fee --rpc-url https://eth.llamarpc.com
```

### find-block

Get the block number closest to the provided timestamp.

```bash
cast find-block <TIMESTAMP> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
# Find block at Unix timestamp
cast find-block 1700000000 --rpc-url https://eth.llamarpc.com
```

## Common Use Cases

1. **Get current block number** for tracking chain state
2. **Find historical blocks** by timestamp for data analysis
3. **Check base fee** for gas price estimation
4. **Retrieve block details** for transaction context
