---
name: test-machines
description: Test Machine contract addresses from makina-test-integrations repo. Use when you need addresses for deployed test Machines (tstUSDC1-5, tstETH1), their Calibers, or accounting tokens. Can fetch latest addresses from GitHub.
allowed-tools: Read, Grep, Glob, Bash, WebFetch
---

# Test Machines

Test Machine deployments from the makina-test-integrations repository.

## Available Test Machines

Look them up from: https://github.com/MakinaHQ/makina-test-integrations/tree/main/machines

## Fetching Latest Addresses

### Examples via curl
```bash
# Fetch config for a specific machine
curl -s https://raw.githubusercontent.com/MakinaHQ/makina-test-integrations/main/machines/tstUSDC1/config.toml

# Extract caliber address for hub/mainnet
curl -s https://raw.githubusercontent.com/MakinaHQ/makina-test-integrations/main/machines/tstUSDC1/config.toml | grep -A1 "\[calibers.mainnet\]" | grep address
```

## Config File Structure

Each machine has a `config.toml` at:
```
machines/{machine-name}/config.toml
```

**Format:**
```toml
name = "tstUSDC1"
chain = "mainnet"
address = "0x..."  # Machine contract address

[calibers.mainnet]
address = "0x..."  # Caliber contract address
swapper = "0x..."  # SwapModule address
rootfiles = "github:MakinaHQ/makina-test-integrations/machines/{name}/mainnet/rootfiles"

[calibers.mainnet.accounting_token]
address = "0x..."
decimals = 6
name = "USD Coin"
symbol = "USDC"

# Multi-chain machines also have:
[calibers.arbitrum]
address = "0x..."
# ...
```

## Example repository Structure

```
makina-test-integrations/
├── machines/
│   ├── <machine token name>/
│   │   ├── config.toml
│   │   └── mainnet/
│   │       ├── caliber.yaml
│   │       ├── instructions/
│   │       └── rootfiles/
│   ├── tstUSDC1/
│   ├── tstUSDC2/
│   ├── tstUSDC3/
│   ├── tstETH1/
│   └── tstUSDC5/
│       ├── config.toml
│       ├── mainnet/
│       └── arbitrum/     # Multi-chain
├── blueprints/           # DeFi protocol interaction blueprints
├── instructions/         # DeFi protocol interaction integration scripts (they use blueprints)
└── token-lists/          # Token data
```

## Common Queries

### Get Machine share price
```bash
cast call <machine addess> "convertToAssets(uint256)(uint256)" 1000000000000000000 --rpc-url https://eth.llamarpc.com
```

### Get Machine total AUM
```bash
cast call <machine addess> "lastTotalAum()(uint256)" --rpc-url https://eth.llamarpc.com
```

### Get Caliber positions
```bash
cast call <caliber address> "getDetailedAum()(uint256,bytes[],bytes[])" --rpc-url https://eth.llamarpc.com
```
