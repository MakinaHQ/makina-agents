# 4byte Commands

Commands for looking up function and event signatures using the openchain.xyz signature database.

## Commands

### 4byte

Get the function signatures for the given selector from openchain.xyz.

```bash
cast 4byte <SELECTOR>
```

**Example:**
```bash
cast 4byte 0xa9059cbb
# Returns: transfer(address,uint256)
```

### 4byte-calldata

Decode calldata by looking up the function selector on openchain.xyz.

```bash
cast 4byte-calldata <CALLDATA>
```

**Example:**
```bash
cast 4byte-calldata 0xa9059cbb000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045000000000000000000000000000000000000000000000000000000000001e240
```

### 4byte-event

Get the event signature for a given topic 0 from openchain.xyz.

```bash
cast 4byte-event <TOPIC>
```

**Example:**
```bash
cast 4byte-event 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
# Returns: Transfer(address,address,uint256)
```

## Notes

- These commands query the openchain.xyz public database
- Not all selectors/topics may have known signatures
- Multiple signatures can share the same selector (collisions)
- Useful for reverse-engineering unknown contract interactions
