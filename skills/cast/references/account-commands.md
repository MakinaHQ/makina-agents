# Account Commands

Commands for querying account information on the blockchain.

## Commands

### balance

Get the balance of an account in wei.

```bash
cast balance <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag (latest, pending, earliest)
- `-e, --ether` - Display balance in ether instead of wei

**Examples:**
```bash
# Get balance in wei
cast balance 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 --rpc-url https://eth.llamarpc.com

# Get balance in ether
cast balance 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -e --rpc-url https://eth.llamarpc.com

# Get balance at specific block
cast balance 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -B 18000000 --rpc-url https://eth.llamarpc.com
```

### nonce

Get the nonce for an account.

```bash
cast nonce <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

**Example:**
```bash
cast nonce 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 --rpc-url https://eth.llamarpc.com
```

### admin

Fetch the EIP-1967 admin account for a proxy contract.

```bash
cast admin <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

**Example:**
```bash
cast admin 0x... --rpc-url https://eth.llamarpc.com
```

### implementation

Fetch the EIP-1967 implementation address for a proxy contract.

```bash
cast implementation <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

**Example:**
```bash
cast implementation 0x... --rpc-url https://eth.llamarpc.com
```

## Common Use Cases

1. **Check account balance** before sending transactions
2. **Get nonce** for manually constructing transactions
3. **Inspect proxy contracts** to find admin and implementation addresses
