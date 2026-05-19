---
name: makina-lite-contracts
description: MakinaLite smart contract interfaces and ABIs. Use when working with MakinaLite - understanding MakinaLiteModule, Safe-module integration, Weiroll instructions, swap/bridge/flash-loan components, OracleRegistry, bridge encoders (Across V4, CCTP V2, LayerZero V2), or the MakinaLiteRegistry/ModuleFactory infra.
allowed-tools: Read, Grep, Glob
---

# MakinaLite Smart Contracts

Interface ABIs and documentation for interacting with MakinaLite - a lightweight version of the Makina protocol that executes DeFi strategies through a [Safe](https://safe.global/) multisig acting as a Safe module.

## Overview

Each MakinaLite deployment is a `MakinaLiteModule` clone installed on a Safe. The module executes pre-approved Weiroll instructions, swaps, bridges, and flash loans on the Safe's behalf. The Safe retains full authority over configuration and risk parameters; an optional lockdown mode enforces value-loss limits, bridge recipient whitelisting, and bridge route/OFT registration checks. Instruction Merkle proof verification is always enforced.

## Architecture

```
┌────────────────────────── Per Safe ──────────────────────────┐
│                                                              │
│   Safe (owner)                                               │
│     └── installs ──► MakinaLiteModule (ERC-1167 clone)       │
│                       ├── WeirollComponent (positions)       │
│                       ├── SwapComponent (DEX aggregators)    │
│                       ├── BridgeComponent (cross-chain)      │
│                       └── OracleRegistry (Chainlink feeds)   │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────── Shared Infra ────────────────────────┐
│   MakinaLiteRegistry  ◄── points to ──┐                      │
│     ├── ModuleFactory ────────── deploys clones              │
│     ├── module implementation                                │
│     ├── fee collector                                        │
│     ├── FlashLoanModule (Morpho)                             │
│     └── Bridge Encoders                                      │
│           ├── AcrossV4BridgeEncoder                          │
│           ├── CctpV2BridgeEncoder                            │
│           └── LayerZeroV2BridgeEncoder                       │
└──────────────────────────────────────────────────────────────┘
```

## Reference Files

| Category | Reference | Description |
|----------|-----------|-------------|
| Core Module | `references/core-module.md` | `MakinaLiteModule`, governance roles, lifecycle |
| Module Components | `references/module-components.md` | Weiroll, Swap, Bridge, Oracle components |
| Bridge Encoders | `references/bridge-encoders.md` | Across V4, CCTP V2, LayerZero V2 |
| Flash Loans | `references/flash-loans.md` | `FlashLoanModule` + Morpho callback |
| Factory & Registry | `references/factory-registry.md` | `ModuleFactory`, `MakinaLiteRegistry` |

## Key Concepts

- **MakinaLiteModule**: Safe module that composes Weiroll, Swap, Bridge, and Oracle components.
- **Safe**: Custody of all assets; sole authority over module configuration. The module pulls inputs from the Safe and forwards outputs back within the same call — it isn't meant to hold balances between operations.
- **Weiroll Instructions**: Pre-approved command chains stored in a Merkle tree; the leaf hash uses a `stateBitmap` to fix some state slots while leaving others variable.
- **Instruction Types**: `MANAGEMENT`, `ACCOUNTING`, `HARVEST`, `FLASHLOAN_MANAGEMENT`.
- **Lockdown Mode**: Enforces additional safety checks (value loss limits, bridge recipient whitelisting, route/OFT registration).
- **Accounting Currency**: Optional per-module currency for position values; defaults to the OracleRegistry's reference currency (`address(0)`).
- **Bridge Encoder**: Per-protocol contract that returns approval/execution targets, value, and calldata for a `BridgeOrder`.
- **Flash Loans**: Routed through `FlashLoanModule` against Morpho; used inside `FLASHLOAN_MANAGEMENT` instructions.

## Module Roles (IMakinaLiteGovernable)

The module uses simple address-mapping roles, distinct from Makina Core terminology:

| Role | Function | Description |
|------|----------|-------------|
| **Safe** | `safe()` | Ultimate owner of the module; sole authority over configuration, risk parameters, and lockdown mode. |
| **Provider** | `provider()` | MakinaLite service account; sets swap fee rate, can `suspend()`/`unsuspend()`. |
| **Operator** | `isOperator(addr)` | Executes strategy actions: account, manage, swap, harvest, bridge. |
| **Guardian** | `isGuardian(addr)` | Emergency `pause()`/`unpause()`. The Safe is always a guardian. |

### Operational States

| State | Set by | Effect |
|-------|--------|--------|
| **Paused** (`paused()`) | Guardian | Blocks all operator actions. |
| **Suspended** (`suspendedByProvider()`) | Provider | Blocks all operator actions. |
| **Lockdown Mode** (`lockdownMode()`) | Safe | Enforces position/swap/bridge value loss limits, bridge recipient whitelisting, and bridge route/OFT registration checks. |

## Infrastructure Roles (AccessManager)

Registry, factory, and bridge encoders are `AccessManagedUpgradeable`. Role IDs are a subset of Makina Core's:

| Role | roleId | Authorized to |
|------|--------|---------------|
| `ADMIN_ROLE` | `0` | Access Manager configuration. |
| `INFRA_CONFIG_ROLE` | `1` | Configure registry and bridge encoders. |
| `STRATEGY_DEPLOYMENT_ROLE` | `2` | Deploy new modules. |
| `INFRA_UPGRADE_ROLE` | `6` | Upgrade infra proxies. |
| `GUARDIAN_ROLE` | `7` | Cancel scheduled AccessManager operations. |

## Common Interactions

```solidity
// Deploy a new module clone (STRATEGY_DEPLOYMENT_ROLE)
address module = IModuleFactory(factory).createModule(initParams, salt, referralKey);

// Operator: account, manage, harvest
IMakinaLiteModule(module).accountForPosition(acctInstruction);
IMakinaLiteModule(module).managePosition(mgmtInstruction, acctInstruction);
IMakinaLiteModule(module).harvest(harvestInstruction, swapOrders);

// Operator: swap (charges swap fee to feeCollector)
IMakinaLiteModule(module).swap(swapOrder);

// Operator: outgoing bridge transfer
IMakinaLiteModule(module).sendOutBridgeTransfer(bridgeOrder);

// Safe calls FlashLoanModule on behalf of the module
IFlashLoanModule(flm).requestFlashLoan(FlashLoanRequest({
    taker: module,
    provider: IFlashLoanModule.FlashLoanProvider.MORPHO,
    instruction: flashloanInstruction,
    token: token,
    amount: amount
}));

// Safe-only configuration
IMakinaLiteModule(module).setAllowedInstrRoot(newRoot);
IMakinaLiteModule(module).setLockdownMode(true);
IMakinaLiteModule(module).setAccountingCurrency(USDC);

// Safe-only recovery
IMakinaLiteModule(module).sweepERC20(token);
IMakinaLiteModule(module).sweepNative();
```

## Common Function Selectors

### Governance / State (IMakinaLiteGovernable)
```
safe()                          0x186f0354
provider()                      0x085d4883
lockdownMode()                  0x12068d44
paused()                        0x5c975abb
suspendedByProvider()           0x41544b92
isOperator(address)             0x6d70f7ae
isGuardian(address)             0x0c68ba21
```

### Module (IMakinaLiteModule)
```
registry()                      0x7b103999
weirollVm()                     0x098ce9f0
allowedInstrRoot()              0x27f99dfe
accountingCurrency()            0x3af89353
maxPositionIncreaseLossBps()    0xcc24fde9
maxPositionDecreaseLossBps()    0x64093da5
maxSwapLossBps()                0x2e607801
swapFeeRate()                   0x3a04801d
sweepERC20(address)             0xe00af4a7
sweepNative()                   0xab803a76
```

### Registry / Factory
```
moduleFactory()                 0x10e72eb2
moduleImplementation()          0x311b0198
feeCollector()                  0xc92cf3c1
flashLoanModule()               0xbcb21ae9
getBridgeEncoder(uint16)        0x2f688174
isMakinaLiteModule(address)     0x0430c9fb
```

## Differences vs. Makina Core

- **No vault/share token**: there is no `Machine`, no `MachineShare`, and no deposit/redeem flow. Assets live in the Safe directly.
- **No cross-chain coordination**: there are no Calibers, CaliberMailboxes, wormhole accounting, or hub/spoke topology — only outgoing bridge transfers.
- **Different governance vocabulary**: `Safe`/`Provider`/`Operator`/`Guardian` instead of `Mechanic`/`RiskManager`/`SecurityCouncil`/`AccountingAgent`.
- **Lockdown mode** is the analogue of restricted operating modes in Core, but applies value-loss checks against the Safe's token balances rather than via a `Machine`-level AUM accounting.
- **Flash loan provider**: Morpho only (other entries in `FlashLoanProvider` are deprecated and preserved for enum stability).

## Sources

- https://github.com/MakinaHQ/makina-lite

## Version

This documentation is current as of:
- **makina-lite**: `3d6741e` - May 18, 2026
- Solidity: `0.8.35`, EVM version: `prague`
