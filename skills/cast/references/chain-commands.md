# Chain Commands

Commands for querying information about the blockchain network.

## Commands

### chain

Get the symbolic name of the current chain.

```bash
cast chain [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
cast chain --rpc-url https://eth.llamarpc.com
# Returns: mainnet

cast chain --rpc-url https://arbitrum.llamarpc.com
# Returns: arbitrum
```

### chain-id

Get the Ethereum chain ID.

```bash
cast chain-id [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
cast chain-id --rpc-url https://eth.llamarpc.com
# Returns: 1

cast chain-id --rpc-url https://polygon-rpc.com
# Returns: 137
```

### client

Get the current client version.

```bash
cast client [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint

**Example:**
```bash
cast client --rpc-url https://eth.llamarpc.com
# Returns: Geth/v1.x.x-...
```

## Common Chain IDs

| Chain | ID |
|-------|-----|
| Ethereum Mainnet | 1 |
| Goerli | 5 |
| Sepolia | 11155111 |
| Polygon | 137 |
| Arbitrum One | 42161 |
| Optimism | 10 |
| Base | 8453 |
| BSC | 56 |
| Avalanche C-Chain | 43114 |

## Common Use Cases

1. **Verify network connection** before sending transactions
2. **Get chain ID** for transaction signing
3. **Check node version** for compatibility
