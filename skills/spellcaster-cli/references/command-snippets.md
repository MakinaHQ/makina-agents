# Spellcaster Command Snippets

Use these patterns for users operating the installed `spellcaster` binary.

## Preflight

```bash
spellcaster --version
spellcaster --help
```

If config file is missing:

```bash
mkdir -p ~/.config/spellcaster
cp example.config.toml ~/.config/spellcaster/config.toml
```

## Interactive Mode

```bash
spellcaster
```

## Non-Interactive Patterns

Display balances:

```bash
spellcaster --machine DUSD --caliber base display-balances
```

Swap tokens:

```bash
spellcaster --machine DUSD --caliber base swap-tokens \
  --input-address 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 \
  --output-address 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 \
  --input-amount 1000000 \
  --slippage 0.5
```

Encode swap:

```bash
spellcaster --machine DUSD --caliber base encode-swap \
  --input-address 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 \
  --output-address 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 \
  --input-amount 1000000 \
  --slippage 0.5
```

Manage position:

```bash
spellcaster --machine DUSD --caliber base manage-position \
  --protocol AaveV3 \
  --action deposit \
  --token USDC \
  --inputs 0x00000000000000000000000000000000000000000000000000000000000f4240
```

Bridge to spoke caliber:

```bash
spellcaster --machine DUSD --caliber base transfer-to-spoke-caliber \
  --destination arbitrum \
  --token 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 \
  --amount 1000000 \
  --slippage 0.5
```

Full AUM update cycle (no caliber required):

```bash
spellcaster --machine DUSD full-aum-update-cycle
```

## Global Flags

- `--config <path>`: override config path.
- `--dev`: developer mode (Tenderly virtual testnets); requires `[dev_rpc_urls]`.
- `--unsigned`: force unsigned transactions.
- `--safe <address>`: submit transactions via Safe address.

## Fast Error Mapping

- `must specify machine in non interactive mode '--machine <NAME>'`
  - Add `--machine <NAME>`.
- `must specify caliber in non interactive mode '--caliber <CHAIN>'`
  - Add `--caliber <CHAIN>` unless running `full-aum-update-cycle`.
- `please set [dev_rpc_urls] in your config to run in dev mode`
  - Add `[dev_rpc_urls]` in config or remove `--dev`.
- `machine '<NAME>' is not in config`
  - Add/update `[machines]` mapping in config.

## Discovery

Use command help for exact flags and subcommand docs:

```bash
spellcaster <subcommand> --help
```
