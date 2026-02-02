# Cryptographic Commands

Commands for hashing data using Ethereum's cryptographic functions.

## Commands

### keccak

Hash arbitrary data using Keccak-256 (the hash function used by Ethereum).

```bash
cast keccak <DATA>
```

**Examples:**
```bash
# Hash a hex string
cast keccak 0x1234
# Returns: 0x56570de287d73cd1cb6092bb8fdee6173974955fdef345ae579ee9f475ea7432

# Hash a string (prefix with "text:")
cast keccak "text:hello"
# Returns: 0x1c8aff950685c2ed4bc3174f3472287b56d9517b9c948127319a09a7a36deac8

# Hash function signature to get selector
cast keccak "transfer(address,uint256)"
# Returns first 4 bytes are the selector
```

### hash-message

Hash a message according to EIP-191 (Ethereum signed message format).

```bash
cast hash-message <MESSAGE>
```

This prepends `"\x19Ethereum Signed Message:\n" + len(message)` before hashing, which is the standard for personal_sign.

**Example:**
```bash
cast hash-message "Hello, World!"
# Returns: 0x...
```

## Technical Details

### Keccak-256

- Keccak-256 is the hash function used throughout Ethereum
- Different from SHA-3 (NIST standardized a modified version)
- Produces a 32-byte (256-bit) hash
- Used for: addresses, storage slots, function selectors, event topics

### EIP-191 Message Hashing

The EIP-191 format prevents signed data from being valid transactions:
```
\x19Ethereum Signed Message:\n<length><message>
```

This is what MetaMask and other wallets use for `personal_sign`.

## Common Use Cases

1. **Generate function selectors** - First 4 bytes of `keccak(signature)`
2. **Compute storage slots** - For mappings: `keccak(key . slot)`
3. **Verify signed messages** - Hash message with EIP-191, then recover signer
4. **Create unique identifiers** - Hash data to create deterministic IDs

## Related Commands

- `cast sig` - Get function selector directly (uses keccak internally)
- `cast sig-event` - Get event topic (uses keccak internally)
- `cast index` - Compute storage slots (uses keccak internally)
