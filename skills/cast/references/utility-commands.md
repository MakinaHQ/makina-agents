# Utility Commands

Various utility commands for Ethereum development.

## Storage Commands

### storage

Get the raw value of a contract's storage slot.

```bash
cast storage <ADDRESS> <SLOT> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

**Examples:**
```bash
# Read storage slot 0
cast storage 0xcontract 0 --rpc-url https://eth.llamarpc.com

# Read specific slot
cast storage 0xcontract 0x0000000000000000000000000000000000000000000000000000000000000001 --rpc-url https://eth.llamarpc.com
```

### storage-root

Get the storage root for an account.

```bash
cast storage-root <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

### index

Compute the storage slot for an entry in a mapping.

```bash
cast index <KEY_TYPE> <KEY> <SLOT>
```

**Examples:**
```bash
# For mapping(address => uint256) at slot 0
cast index address 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 0

# For mapping(uint256 => uint256) at slot 1
cast index uint256 42 1
```

### index-erc7201

Compute storage slots as specified by ERC-7201 (Namespaced Storage Layout).

```bash
cast index-erc7201 <ID>
```

**Example:**
```bash
cast index-erc7201 "example.main"
```

### proof

Generate a storage proof for a given storage slot.

```bash
cast proof <ADDRESS> [SLOTS]... [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

**Example:**
```bash
cast proof 0xcontract 0 1 2 --rpc-url https://eth.llamarpc.com
```

## Selector & Signature Commands

### sig

Get the selector for a function.

```bash
cast sig <SIG>
```

**Examples:**
```bash
cast sig "transfer(address,uint256)"
# Returns: 0xa9059cbb

cast sig "balanceOf(address)"
# Returns: 0x70a08231
```

### sig-event

Generate event signatures (topic 0) from event string.

```bash
cast sig-event <EVENT>
```

**Examples:**
```bash
cast sig-event "Transfer(address,address,uint256)"
# Returns: 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef

cast sig-event "Approval(address,address,uint256)"
```

### selectors

Extract function selectors and arguments from bytecode.

```bash
cast selectors <BYTECODE>
```

**Example:**
```bash
cast selectors 0x6080604052...
```

### upload-signature

Upload the given signatures to openchain.xyz.

```bash
cast upload-signature <SIGNATURES>...
```

**Example:**
```bash
cast upload-signature "transfer(address,uint256)" "approve(address,uint256)"
```

## CREATE2 & Contract Deployment

### create2

Generate a deterministic contract address using CREATE2.

```bash
cast create2 [OPTIONS]
```

**Options:**
- `--starts-with <HEX>` - Vanity prefix
- `--ends-with <HEX>` - Vanity suffix
- `--deployer <ADDRESS>` - Deployer address
- `--init-code <CODE>` - Contract init code
- `--init-code-hash <HASH>` - Init code hash
- `--salt <SALT>` - Use specific salt

**Examples:**
```bash
# Find salt for vanity address
cast create2 --starts-with dead --deployer 0x... --init-code-hash 0x...

# Calculate address for known salt
cast create2 --salt 0x... --deployer 0x... --init-code-hash 0x...
```

### artifact

Generate an artifact file that can be used to deploy a contract locally.

```bash
cast artifact <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-e, --etherscan-api-key <KEY>` - Etherscan API key

### constructor-args

Display constructor arguments used for contract initialization.

```bash
cast constructor-args <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-e, --etherscan-api-key <KEY>` - Etherscan API key

## Interface & Bindings

### interface

Generate a Solidity interface from a given ABI.

```bash
cast interface <ADDRESS_OR_ABI> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint (if using address)
- `-e, --etherscan-api-key <KEY>` - Etherscan API key
- `-n, --name <NAME>` - Interface name
- `-o, --output <PATH>` - Output file

**Example:**
```bash
cast interface 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 -e $ETHERSCAN_API_KEY
```

### bind

Generate Rust bindings from a given ABI.

```bash
cast bind <ADDRESS_OR_ABI> [OPTIONS]
```

## Bitwise Operations

### shl

Perform a left shifting operation.

```bash
cast shl <VALUE> <BITS>
```

**Example:**
```bash
cast shl 1 8
# Returns: 256 (1 << 8)
```

### shr

Perform a right shifting operation.

```bash
cast shr <VALUE> <BITS>
```

**Example:**
```bash
cast shr 256 8
# Returns: 1 (256 >> 8)
```

### pad

Pad hex data to a specified length.

```bash
cast pad [OPTIONS] <DATA>
```

**Options:**
- `--left` - Left pad (default)
- `--right` - Right pad
- `-s, --size <SIZE>` - Pad to size (default: 32)

## Constants

### address-zero

Print the zero address.

```bash
cast address-zero
# Returns: 0x0000000000000000000000000000000000000000
```

### hash-zero

Print the zero hash.

```bash
cast hash-zero
# Returns: 0x0000000000000000000000000000000000000000000000000000000000000000
```

### max-int

Print the maximum value of the given integer type.

```bash
cast max-int [TYPE]
```

**Example:**
```bash
cast max-int int256
# Returns: 57896044618658097711785492504343953926634992332820282019728792003956564819967
```

### max-uint

Print the maximum value of the given unsigned integer type.

```bash
cast max-uint [TYPE]
```

**Example:**
```bash
cast max-uint uint256
# Returns: 115792089237316195423570985008687907853269984665640564039457584007913129639935
```

### min-int

Print the minimum value of the given integer type.

```bash
cast min-int [TYPE]
```

**Example:**
```bash
cast min-int int256
```

## Other Utilities

### source

Get the source code of a contract from a block explorer.

```bash
cast source <ADDRESS> [OPTIONS]
```

**Options:**
- `-e, --etherscan-api-key <KEY>` - Etherscan API key
- `--chain <CHAIN>` - Chain name
- `-d, --directory <DIR>` - Output directory

**Example:**
```bash
cast source 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 -e $ETHERSCAN_API_KEY
```

### rpc

Perform a raw JSON-RPC request.

```bash
cast rpc <METHOD> [PARAMS]... [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-r, --raw` - Don't parse result

**Examples:**
```bash
cast rpc eth_blockNumber --rpc-url https://eth.llamarpc.com

cast rpc eth_getBalance 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 latest --rpc-url https://eth.llamarpc.com
```

### completions

Generate shell completions script.

```bash
cast completions <SHELL>
```

**Shells:** bash, zsh, fish, powershell, elvish

**Example:**
```bash
cast completions zsh > ~/.zsh/completions/_cast
```

### trace

Trace a transaction or call.

```bash
cast trace [OPTIONS]
```

See `cast trace --help` for full options.
