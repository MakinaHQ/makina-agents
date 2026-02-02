# ENS Commands

Commands for interacting with the Ethereum Name Service (ENS).

## Commands

### resolve-name

Perform an ENS lookup to resolve a name to an address.

```bash
cast resolve-name <NAME> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `--verify` - Verify the resolved address

**Examples:**
```bash
cast resolve-name vitalik.eth --rpc-url https://eth.llamarpc.com
# Returns: 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045

cast resolve-name uniswap.eth --rpc-url https://eth.llamarpc.com
```

### lookup-address

Perform an ENS reverse lookup to get the name for an address.

```bash
cast lookup-address <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `--verify` - Verify the reverse record

**Example:**
```bash
cast lookup-address 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url https://eth.llamarpc.com
# Returns: vitalik.eth
```

### namehash

Calculate the ENS namehash of a name.

```bash
cast namehash <NAME>
```

**Examples:**
```bash
cast namehash eth
# Returns: 0x93cdeb708b7545dc668eb9280176169d1c33cfd8ed6f04690a0bcc88a93fc4ae

cast namehash vitalik.eth
```

## How ENS Works

ENS uses a hierarchical naming system similar to DNS:
- Names are hashed using the namehash algorithm
- The hash is used to look up records in the ENS registry
- Records can store addresses, content hashes, text records, and more

## Common Use Cases

1. **Resolve human-readable names** to addresses before sending transactions
2. **Reverse lookup** to display names instead of addresses in UIs
3. **Calculate namehash** for direct interaction with ENS contracts
4. **Verify ENS records** to ensure bidirectional resolution

## Notes

- ENS only works on Ethereum mainnet (and some testnets)
- Reverse records must be set by the address owner
- Some names may not have a primary name set for reverse lookup
