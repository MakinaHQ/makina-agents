# Code Commands

Commands for inspecting smart contract bytecode.

## Commands

### code

Get the runtime bytecode of a contract.

```bash
cast code <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag
- `--disassemble` - Disassemble the bytecode

**Examples:**
```bash
# Get contract bytecode
cast code 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 --rpc-url https://eth.llamarpc.com

# Get bytecode at specific block
cast code 0x... -B 18000000 --rpc-url https://eth.llamarpc.com
```

### codehash

Get the codehash for an account.

```bash
cast codehash <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

**Example:**
```bash
cast codehash 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 --rpc-url https://eth.llamarpc.com
```

### codesize

Get the runtime bytecode size of a contract.

```bash
cast codesize <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-B, --block <BLOCK>` - Block number or tag

**Example:**
```bash
cast codesize 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 --rpc-url https://eth.llamarpc.com
```

### creation-code

Download a contract creation code from Etherscan and RPC.

```bash
cast creation-code <ADDRESS> [OPTIONS]
```

**Options:**
- `--rpc-url <URL>` - RPC endpoint
- `-e, --etherscan-api-key <KEY>` - Etherscan API key

**Example:**
```bash
cast creation-code 0x... --rpc-url https://eth.llamarpc.com -e $ETHERSCAN_API_KEY
```

### disassemble

Disassemble hex-encoded bytecode into human-readable representation.

```bash
cast disassemble <BYTECODE>
```

**Example:**
```bash
cast disassemble 0x6080604052...

# Or pipe from code command
cast code 0x... --rpc-url https://eth.llamarpc.com | cast disassemble
```

## Common Use Cases

1. **Verify contract deployment** by comparing bytecode
2. **Check if address is a contract** (EOAs have empty code)
3. **Analyze contract bytecode** for security research
4. **Get creation code** for deploying identical contracts
