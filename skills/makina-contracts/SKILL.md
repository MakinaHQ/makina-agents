---
name: makina-contracts
description: Makina smart contract interfaces and ABIs. Use when working with Makina protocol - understanding Machine, Caliber, Registry interfaces, function signatures, structs, events, deposit/redeem flows, or cross-chain bridge operations.
allowed-tools: Read, Grep, Glob
---

# Makina Smart Contracts

Interface ABIs and documentation for interacting with Makina smart contracts - a cross-chain investment strategy protocol.

## Overview

Makina is a cross-chain DeFi protocol enabling coordinated investment strategies across Hub and Spoke blockchains:

- **makina-core**: Core infrastructure (Machine, Caliber, Registries, Bridges)
- **makina-periphery**: Peripheral contracts (Depositors, Redeemers, Fees, Security)

## Architecture

```
Hub Chain (Ethereum)                Spoke Chains (Arbitrum, Base)
┌─────────────────────────────┐     ┌─────────────────────────────┐
│  Machine                    │     │  CaliberMailbox             │
│  ├── HubCaliber            │◄────►│  └── SpokeCaliber          │
│  ├── MachineShare (ERC20)  │     │                             │
│  └── PreDepositVault       │     │  Bridge Adapters            │
│                             │     │  ├── Across V3             │
│  Periphery                  │     │  └── LayerZero V2          │
│  ├── DirectDepositor       │     └─────────────────────────────┘
│  ├── AsyncRedeemer         │
│  ├── SecurityModule        │
│  └── WatermarkFeeManager   │
└─────────────────────────────┘
```

## Reference Files

### Core Contracts

| Category | Reference | Description |
|----------|-----------|-------------|
| Machine & Caliber | `references/core-machine.md` | Main vault and strategy execution |
| Registries | `references/core-registries.md` | Token, Oracle, Chain registries |
| Factories | `references/core-factories.md` | Contract deployment |
| Bridge | `references/core-bridge.md` | Cross-chain adapters |
| Swap & Utils | `references/core-swap-utils.md` | Swap modules, utilities |

### Periphery Contracts

| Category | Reference | Description |
|----------|-----------|-------------|
| Deposit & Redeem | `references/periphery-deposit-redeem.md` | User flows |
| Security Module | `references/periphery-security.md` | Slashing, cooldowns |
| Fee Management | `references/periphery-fees.md` | Fee calculation |
| Oracles | `references/periphery-oracles.md` | Share price oracles |
| Registry & Factory | `references/periphery-registry-factory.md` | Periphery infra |

## Key Concepts

- **Machine**: Main vault on hub chain, manages deposits/redemptions/shares
- **Caliber**: Strategy execution, manages positions per chain
- **MachineShare**: ERC20 representing vault ownership
- **Bridge Adapters**: Cross-chain transfers (Across V3, LayerZero V2)
- **Security Module**: Staked shares as protocol insurance

## Governance Roles (IMakinaGovernable)

Makina uses specific terminology for governance roles:

| Role | Function | Description |
|------|----------|-------------|
| **Mechanic** | `mechanic()` | The operator/admin who manages day-to-day operations (execute strategies, manage positions). This is the "operator" in Makina terminology. |
| **Risk Manager** | `riskManager()` | Sets risk parameters and thresholds |
| **Security Council** | `securityCouncil()` | Emergency actions, can enable recovery mode |
| **Authority** | `authority()` | Access control contract for role-based permissions |
| **Accounting Agent** | `isAccountingAgent(addr)` | Permissioned addresses that can perform accounting operations when restricted accounting mode is enabled |

### Restricted Accounting Mode (v1.1.0+)

When `restrictedAccountingMode()` is true, only designated Accounting Agents can call accounting functions. Use `isAccountingAuthorized(addr)` to check if an address can perform accounting.

```solidity
// Check if user can perform accounting
bool canAccount = IMakinaGovernable(machine).isAccountingAuthorized(user);

// Add/remove accounting agents (security council only)
IMakinaGovernable(machine).addAccountingAgent(agent);
IMakinaGovernable(machine).removeAccountingAgent(agent);
```

## Share Price Guards (v1.1.0+)

The Machine enforces a maximum share price change rate to protect against manipulation:

```solidity
// View max allowed change rate (1e18 = 100% per second)
uint256 maxRate = IMachine(machine).maxSharePriceChangeRate();

// Set new rate (risk manager only)
IMachine(machine).setMaxSharePriceChangeRate(newRate);
```

## Common Interactions

```solidity
// Deposit
IDirectDepositor(depositor).deposit(assets, receiver, minShares, referralKey);

// Redeem (async)
uint256 requestId = IAsyncRedeemer(redeemer).requestRedeem(shares, receiver, minAssets);
IAsyncRedeemer(redeemer).finalizeRequests(upToRequestId, minAssets);
IAsyncRedeemer(redeemer).claimAssets(requestId);

// Read share price
uint256 price = IMachineShareOracle(oracle).getSharePrice();
uint256 assets = IMachine(machine).convertToAssets(1e18);

// Lock in Security Module
uint256 shares = ISecurityModule(sm).lock(assets, receiver, minShares);
```

## Common Function Selectors

Useful when making raw JSON-RPC calls or encoding calldata.

### Governance (IMakinaGovernable)
```
mechanic()                    0xc549beec
riskManager()                 0x47842663
securityCouncil()             0x27eb6c0f
authority()                   0xbf7e214f
recoveryMode()                0x07a00b1f
restrictedAccountingMode()    0x8a6e2b7c
isAccountingAgent(address)    0x5c975abb
isAccountingAuthorized(address) 0x9b3ba79f
```

### Machine (IMachine)
```
depositor()                   0xc7c4ff46
redeemer()                    0x2ba29d38
shareToken()                  0x6c9fa59e
accountingToken()             0xda68cf8b
hubCaliber()                  0x34a41188
feeManager()                  0xd0fb0203
lastTotalAum()                0x74c59381
shareLimit()                  0x3be75aa3
maxSharePriceChangeRate()     0x1a2e9b4d
convertToShares(uint256)      0xc6e6f592
convertToAssets(uint256)      0x07a2d13a
```

### Caliber (ICaliber)
```
getDetailedAum()              0xd98964f1
getPositionsLength()          0x740c0c44
isBaseToken(address)          0x85bb6a3c
allowedInstrRoot()            0x27f99dfe
isInstrRootGuardian(address)  0x6e8a9c3f
pendingAllowedInstrRoot()     0x4b5d7a2e
```

## Sources

- https://github.com/MakinaHQ/makina-core
- https://github.com/MakinaHQ/makina-periphery

## Version

This documentation is current as of:
- **makina-core**: v1.1.0 (`ff6f036`) - January 26, 2025
- **makina-periphery**: v1.1.0 (`e73532b`) - January 26, 2025
