# Conversion Commands

Commands for converting between various data formats used in Ethereum.

## Wei/Ether Conversions

### from-wei

Convert wei into an ETH amount.

```bash
cast from-wei <VALUE> [UNIT]
```

**Units:** wei, gwei, ether (default: ether)

**Examples:**
```bash
cast from-wei 1000000000000000000
# Returns: 1.000000000000000000

cast from-wei 1000000000 gwei
# Returns: 1.000000000
```

### to-wei

Convert an ETH amount to wei.

```bash
cast to-wei <VALUE> [UNIT]
```

**Examples:**
```bash
cast to-wei 1
# Returns: 1000000000000000000

cast to-wei 1 gwei
# Returns: 1000000000
```

### to-unit

Convert an ETH amount into another unit (ether, gwei, or wei).

```bash
cast to-unit <VALUE> <UNIT>
```

**Example:**
```bash
cast to-unit 1000000000000000000wei ether
```

## Hex Conversions

### from-bin

Convert binary data into hex data.

```bash
cast from-bin
```

Reads from stdin.

### to-hex

Convert a number of one base to hexadecimal.

```bash
cast to-hex <VALUE>
```

**Example:**
```bash
cast to-hex 255
# Returns: 0xff
```

### to-dec

Convert a number of one base to decimal.

```bash
cast to-dec <VALUE>
```

**Example:**
```bash
cast to-dec 0xff
# Returns: 255
```

### to-base

Convert a number from one base to another.

```bash
cast to-base <VALUE> <BASE>
```

**Example:**
```bash
cast to-base 255 16
# Returns: 0xff
```

### to-hexdata

Normalize input to lowercase, 0x-prefixed hex.

```bash
cast to-hexdata <DATA>
```

**Example:**
```bash
cast to-hexdata "hello"
```

### concat-hex

Concatenate hex strings.

```bash
cast concat-hex <DATA>...
```

**Example:**
```bash
cast concat-hex 0x1234 0x5678
# Returns: 0x12345678
```

## Bytes32 Conversions

### to-bytes32

Right-pad hex data to 32 bytes.

```bash
cast to-bytes32 <DATA>
```

**Example:**
```bash
cast to-bytes32 0x1234
```

### format-bytes32-string

Format a string into bytes32 encoding.

```bash
cast format-bytes32-string <STRING>
```

**Example:**
```bash
cast format-bytes32-string "hello"
```

### parse-bytes32-string

Parse a string from bytes32 encoding.

```bash
cast parse-bytes32-string <BYTES32>
```

**Example:**
```bash
cast parse-bytes32-string 0x68656c6c6f000000000000000000000000000000000000000000000000000000
# Returns: hello
```

### parse-bytes32-address

Parse a checksummed address from bytes32 encoding.

```bash
cast parse-bytes32-address <BYTES32>
```

## Integer Conversions

### to-int256

Convert a number to a hex-encoded int256.

```bash
cast to-int256 <VALUE>
```

**Example:**
```bash
cast to-int256 -1
```

### to-uint256

Convert a number to a hex-encoded uint256.

```bash
cast to-uint256 <VALUE>
```

**Example:**
```bash
cast to-uint256 1000
```

## Fixed Point Conversions

### from-fixed-point

Convert a fixed point number into an integer.

```bash
cast from-fixed-point <DECIMALS> <VALUE>
```

**Example:**
```bash
cast from-fixed-point 18 1.5
# Returns: 1500000000000000000
```

### to-fixed-point

Convert an integer into a fixed point number.

```bash
cast to-fixed-point <DECIMALS> <VALUE>
```

**Example:**
```bash
cast to-fixed-point 18 1500000000000000000
# Returns: 1.5
```

### format-units

Format a number from smallest unit to decimal with arbitrary decimals.

```bash
cast format-units <VALUE> [DECIMALS]
```

**Example:**
```bash
cast format-units 1000000 6
# Returns: 1.0 (USDC has 6 decimals)
```

### parse-units

Convert a number from decimal to smallest unit with arbitrary decimals.

```bash
cast parse-units <VALUE> [DECIMALS]
```

**Example:**
```bash
cast parse-units 1.5 18
# Returns: 1500000000000000000
```

## RLP Conversions

### from-rlp

Decode RLP hex-encoded data.

```bash
cast from-rlp <DATA>
```

### to-rlp

RLP encode hex data, or an array of hex data.

```bash
cast to-rlp <DATA>
```

## String/UTF8 Conversions

### from-utf8

Convert UTF8 text to hex.

```bash
cast from-utf8 <TEXT>
```

**Example:**
```bash
cast from-utf8 "hello"
# Returns: 0x68656c6c6f
```

### to-utf8

Convert hex data to a UTF-8 string.

```bash
cast to-utf8 <HEX>
```

**Example:**
```bash
cast to-utf8 0x68656c6c6f
# Returns: hello
```

### to-ascii

Convert hex data to an ASCII string.

```bash
cast to-ascii <HEX>
```

## Address Conversion

### to-check-sum-address

Convert an address to checksummed format (EIP-55).

```bash
cast to-check-sum-address <ADDRESS>
```

**Example:**
```bash
cast to-check-sum-address 0xd8da6bf26964af9d7eed9e03e53415d37aa96045
# Returns: 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

## Common Use Cases

1. **Token amounts** - Convert between human-readable and raw amounts using `format-units`/`parse-units`
2. **ETH conversions** - Use `to-wei`/`from-wei` for ETH amounts
3. **Data encoding** - Use `to-bytes32`, `format-bytes32-string` for storage/calldata
4. **Address validation** - Use `to-check-sum-address` for EIP-55 compliance
