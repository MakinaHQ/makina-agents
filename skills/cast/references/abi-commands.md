# ABI Commands

Commands for encoding and decoding ABI (Application Binary Interface) data used in Ethereum smart contract interactions.

## Commands

### abi-encode

ABI encode the given function argument, excluding the selector.

```bash
cast abi-encode <SIG> [ARGS]...
```

**Example:**
```bash
cast abi-encode "transfer(address,uint256)" 0x1234...5678 1000000
```

### abi-encode-event

ABI encode an event and its arguments to generate topics and data.

```bash
cast abi-encode-event <SIG> [ARGS]...
```

**Example:**
```bash
cast abi-encode-event "Transfer(address indexed,address indexed,uint256)" 0xfrom 0xto 1000
```

### decode-abi

Decode ABI-encoded input or output data.

```bash
cast decode-abi <SIG> <CALLDATA>
```

**Example:**
```bash
cast decode-abi "balanceOf(address)(uint256)" 0x000000000000000000000000000000000000000000000000000000000001e240
```

### decode-calldata

Decode ABI-encoded input data (calldata).

```bash
cast decode-calldata <SIG> <CALLDATA>
```

**Example:**
```bash
cast decode-calldata "transfer(address,uint256)" 0xa9059cbb000000000000000000000000...
```

### decode-error

Decode custom error data from a failed transaction.

```bash
cast decode-error <ERROR_DATA>
```

**Example:**
```bash
cast decode-error 0x08c379a0...
```

### decode-event

Decode event data from transaction logs.

```bash
cast decode-event <SIG> <DATA> [TOPICS]...
```

**Example:**
```bash
cast decode-event "Transfer(address indexed,address indexed,uint256)" 0x... 0xtopic1 0xtopic2
```

### decode-string

Decode an ABI-encoded string.

```bash
cast decode-string <DATA>
```

**Example:**
```bash
cast decode-string 0x0000000000000000000000000000000000000000000000000000000000000020...
```

## Common Use Cases

1. **Preparing transaction data** - Use `abi-encode` to create calldata for transactions
2. **Debugging failed transactions** - Use `decode-error` to understand revert reasons
3. **Parsing event logs** - Use `decode-event` to read emitted events
4. **Inspecting calldata** - Use `decode-calldata` to understand function calls
