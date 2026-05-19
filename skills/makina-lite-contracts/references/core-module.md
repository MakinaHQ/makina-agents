# Core: MakinaLiteModule

The Safe-module entry point. Composes `WeirollComponent`, `SwapComponent`, `BridgeComponent`, and `OracleRegistry`, and inherits governance state from `MakinaLiteGovernable`.

## IMakinaLiteModule

```solidity
interface IMakinaLiteModule is
    IMakinaLiteContext,
    IMakinaLiteGovernable,
    IOracleRegistry,
    IWeirollComponent,
    ISwapComponent,
    IBridgeComponent
{
    struct MakinaLiteModuleInitParams {
        address safe;
        address initialProvider;
        bytes32 initialAllowedInstrRoot;
        uint256 initialMaxPositionIncreaseLossBps;
        uint256 initialMaxPositionDecreaseLossBps;
        uint256 initialMaxSwapLossBps;
        uint256 initialSwapFeeRate; // 1e18 = 100%
    }

    function initialize(MakinaLiteModuleInitParams calldata params) external;

    // Safe-only recovery
    function sweepERC20(address token) external;
    function sweepNative() external;
}
```

Notes:
- Deployed as an ERC-1167 minimal clone via `ModuleFactory`.
- `sweepERC20` / `sweepNative` exist because the module isn't meant to hold balances — they recover anything that lands on the module by mistake (and the LayerZero native gas buffer).

## IMakinaLiteContext

```solidity
interface IMakinaLiteContext {
    function registry() external view returns (address);
}
```

The shared `MakinaLiteRegistry` is the lookup hub for `feeCollector`, `flashLoanModule`, and bridge encoders. See [factory-registry](factory-registry.md).

## IMakinaLiteGovernable

Module-level access control. Distinct from Makina Core's `IMakinaGovernable`.

```solidity
interface IMakinaLiteGovernable {
    event GuardianAdded(address indexed newGuardian);
    event GuardianRemoved(address indexed guardian);
    event LockdownModeChanged(bool indexed enabled);
    event OperatorAdded(address indexed newOperator);
    event OperatorRemoved(address indexed operator);
    event Paused(address indexed guardian);
    event Unpaused(address indexed guardian);
    event ProviderChanged(address indexed oldProvider, address indexed newProvider);
    event Suspended();
    event Unsuspended();

    // Addresses / state
    function safe() external view returns (address);
    function provider() external view returns (address);
    function isOperator(address account) external view returns (bool);
    function isGuardian(address account) external view returns (bool);
    function lockdownMode() external view returns (bool);
    function suspendedByProvider() external view returns (bool);
    function paused() external view returns (bool);

    // Provider
    function setProvider(address newProvider) external;
    function suspend() external;
    function unsuspend() external;

    // Safe
    function addOperator(address newOperator) external;
    function removeOperator(address operator) external;
    function addGuardian(address newGuardian) external;
    function removeGuardian(address guardian) external;
    function setLockdownMode(bool enabled) external;

    // Guardian
    function pause() external;
    function unpause() external;
}
```

### Roles & state transitions

| Role | Capabilities |
|------|--------------|
| **Safe** | Module configuration (allowed-instr root, oracle feeds, loss caps, swapper targets, bridge encoders' lockdown allowlists, lockdown toggle, role membership). Also a permanent guardian. |
| **Provider** | `setSwapFeeRate`, `setProvider`, `suspend`/`unsuspend`. |
| **Operator** | `accountForPosition*`, `managePosition*`, `harvest`, `swap`, `sendOutBridgeTransfer`. Requires `!paused && !suspendedByProvider`. |
| **Guardian** | `pause` / `unpause`. |

Three independent state flags determine whether operator actions can run:

| State | Source | Cleared by |
|-------|--------|------------|
| `paused` | Guardian `pause()` | Guardian `unpause()` |
| `suspendedByProvider` | Provider `suspend()` | Provider `unsuspend()` |
| `lockdownMode` | Safe `setLockdownMode(true)` | Safe `setLockdownMode(false)` |

`paused` and `suspendedByProvider` both gate operator actions. `lockdownMode` does not gate execution — it adds value-loss checks, recipient whitelisting, and bridge route/OFT registration enforcement.

## Lifecycle

1. **Deploy**: `ModuleFactory.createModule(params, salt, referralKey)` deploys the clone and calls `initialize`. The Safe must subsequently enable the module on itself (`enableModule(module)` on the Safe).
2. **Configure** (Safe): set Merkle root, oracle feed routes + staleness thresholds, loss BPS caps, swapper targets, bridge encoder allowlists, accounting currency. Optionally enable lockdown mode.
3. **Operate** (Operator): account/manage/harvest/swap/bridge.
4. **Emergency** (Guardian): `pause()`.
5. **Service controls** (Provider): adjust `swapFeeRate`, or `suspend()` to halt operations.
6. **Recovery** (Safe): `sweepERC20`, `sweepNative` for stray balances.

## Initialization parameter notes

| Param | Notes |
|-------|-------|
| `safe` | Must be the Safe that owns the module. Cannot be changed post-init. |
| `initialProvider` | Service account; can later transfer the role. |
| `initialAllowedInstrRoot` | Merkle root of approved Weiroll instructions. Required even outside lockdown mode. |
| `initialMaxPosition{Increase,Decrease}LossBps` | Loss caps in BPS; enforced only in lockdown mode. Bounded by `_checkBps`. |
| `initialMaxSwapLossBps` | Swap loss cap in BPS; enforced in lockdown mode. |
| `initialSwapFeeRate` | Provider-set rate, `1e18` = 100%. Bounded by `_checkFeeRate`. |
