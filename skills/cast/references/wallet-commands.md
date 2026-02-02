# Wallet Commands

Commands for wallet management and cryptographic operations.

## Commands

### wallet

Wallet management utilities. This is a subcommand group with multiple sub-subcommands.

```bash
cast wallet <SUBCOMMAND>
```

## Subcommands

### wallet new

Create a new random wallet (keypair).

```bash
cast wallet new [OPTIONS]
```

**Options:**
- `-p, --password` - Prompt for password to encrypt keystore
- `--unsafe-password <PASSWORD>` - Password (not recommended)
- `-n, --number <NUM>` - Number of wallets to generate

**Example:**
```bash
cast wallet new
# Returns: address and private key

cast wallet new -n 5
# Generate 5 new wallets
```

### wallet address

Get the address from a private key.

```bash
cast wallet address [OPTIONS]
```

**Options:**
- `--private-key <KEY>` - Private key

**Example:**
```bash
cast wallet address --private-key 0x...
```

### wallet sign

Sign a message with a private key.

```bash
cast wallet sign <MESSAGE> [OPTIONS]
```

**Options:**
- `--private-key <KEY>` - Private key
- `--from <ADDRESS>` - Use keystore for address
- `--no-hash` - Don't hash message (sign raw)

**Examples:**
```bash
# Sign a message (EIP-191)
cast wallet sign "Hello, World!" --private-key 0x...

# Sign without hashing
cast wallet sign 0x... --private-key 0x... --no-hash
```

### wallet verify

Verify a signed message.

```bash
cast wallet verify <MESSAGE> <SIGNATURE> <ADDRESS>
```

**Example:**
```bash
cast wallet verify "Hello, World!" 0xsignature... 0xaddress
```

### wallet import

Import a private key into the keystore.

```bash
cast wallet import <NAME> [OPTIONS]
```

**Options:**
- `--private-key <KEY>` - Private key to import
- `-i, --interactive` - Enter key interactively
- `-k, --keystore-dir <DIR>` - Keystore directory

**Example:**
```bash
cast wallet import my-wallet --private-key 0x... -i
```

### wallet list

List all accounts in the keystore.

```bash
cast wallet list [OPTIONS]
```

**Options:**
- `-k, --keystore-dir <DIR>` - Keystore directory

**Example:**
```bash
cast wallet list
```

### wallet derive-private-key

Derive a private key from a mnemonic phrase.

```bash
cast wallet derive-private-key <MNEMONIC> [INDEX]
```

**Example:**
```bash
cast wallet derive-private-key "word1 word2 ... word12" 0
```

### wallet vanity

Generate a vanity address.

```bash
cast wallet vanity [OPTIONS]
```

**Options:**
- `--starts-with <HEX>` - Prefix to match
- `--ends-with <HEX>` - Suffix to match
- `--nonce <NONCE>` - Starting nonce for search

**Example:**
```bash
cast wallet vanity --starts-with dead
# Finds address starting with 0xdead...
```

### wallet sign-auth

Sign an EIP-7702 authorization.

```bash
cast wallet sign-auth <ADDRESS> [OPTIONS]
```

**Options:**
- `--private-key <KEY>` - Private key
- `--nonce <NONCE>` - Authorization nonce
- `--chain-id <ID>` - Chain ID

## recover-authority

Recover an EIP-7702 authority from an Authorization JSON string.

```bash
cast recover-authority <AUTH_JSON>
```

## Keystore Locations

Default keystore directories:
- Linux: `~/.foundry/keystores`
- macOS: `~/.foundry/keystores`
- Windows: `%USERPROFILE%\.foundry\keystores`

## Security Notes

1. **Never share private keys** - Keep them secure and never commit to version control
2. **Use environment variables** - Store keys in env vars, not command line history
3. **Use hardware wallets** - For production, consider using `--from` with hardware wallet
4. **Encrypt keystores** - Always use passwords for imported keys
5. **Verify before signing** - Always review what you're signing

## Common Use Cases

1. **Generate test wallets** - Use `wallet new` for development
2. **Sign messages** - Use `wallet sign` for off-chain signatures
3. **Vanity addresses** - Use `wallet vanity` for custom addresses
4. **Key management** - Use `wallet import/list` for organized key storage
5. **Derive from mnemonic** - Use `derive-private-key` for HD wallet paths
