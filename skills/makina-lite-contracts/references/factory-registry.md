# Factory & Registry

Shared infrastructure for deploying and locating MakinaLite modules and their bridge encoders. Both contracts are `AccessManagedUpgradeable`.

## IModuleFactory

Deploys new `MakinaLiteModule` instances as [ERC-1167](https://eips.ethereum.org/EIPS/eip-1167) minimal clones using deterministic `CREATE2` salts.

```solidity
interface IModuleFactory {
    event MakinaLiteModuleCreated(
        address indexed module,
        address indexed implementation,
        bytes32 indexed referralKey
    );

    function isMakinaLiteModule(address module) external view returns (bool);

    function createModule(
        IMakinaLiteModule.MakinaLiteModuleInitParams calldata params,
        bytes32 salt,
        bytes32 referralKey
    ) external returns (address);
}
```

Notes:
- `createModule` is gated by `STRATEGY_DEPLOYMENT_ROLE` (roleId `2`).
- `salt == bytes32(0)` reverts (`ZeroSalt`).
- Implementation is read from `MakinaLiteRegistry.moduleImplementation()` at call time, so future deployments use whatever is currently registered.
- The factory tracks every clone it has deployed in `isMakinaLiteModule` — this is the gate `FlashLoanModule` uses to authorize callers. See [flash-loans](flash-loans.md).
- Address can be derived deterministically: `Clones.predictDeterministicAddress(implementation, salt, factory)`.

## IMakinaLiteRegistry

Single upgradeable registry holding addresses of shared infrastructure.

```solidity
interface IMakinaLiteRegistry {
    event BridgeEncoderChanged(uint16 indexed bridgeId, address indexed oldBridgeEncoder, address indexed newBridgeEncoder);
    event FeeCollectorChanged(address indexed oldFeeCollector, address indexed newFeeCollector);
    event FlashLoanModuleChanged(address indexed oldFlashLoanModule, address indexed newFlashLoanModule);
    event ModuleFactoryChanged(address indexed oldModuleFactory, address indexed newModuleFactory);
    event ModuleImplementationChanged(address indexed oldModuleImplementation, address indexed newModuleImplementation);

    function moduleFactory() external view returns (address);
    function moduleImplementation() external view returns (address);
    function feeCollector() external view returns (address);
    function flashLoanModule() external view returns (address);
    function getBridgeEncoder(uint16 bridgeId) external view returns (address);

    function setModuleFactory(address factory) external;
    function setModuleImplementation(address newImplementation) external;
    function setFeeCollector(address newFeeCollector) external;
    function setFlashLoanModule(address newFlashLoanModule) external;
    function setBridgeEncoder(uint16 bridgeId, address bridgeEncoder) external;
}
```

All setters are `restricted` and gated by `INFRA_CONFIG_ROLE` (roleId `1`).

### What modules use it for

- `feeCollector` — destination of swap fees (`SwapComponent._chargeSwapFee`).
- `flashLoanModule` — `manageFlashLoan` rejects calls from any other source.
- `getBridgeEncoder(bridgeId)` — resolved at `sendOutBridgeTransfer` time to encode the protocol-specific call.

The factory and the module implementation registered here are independent: bumping `moduleImplementation` does not migrate existing clones (since they are immutable bytecode clones); only new clones will use the new implementation.

## Upgrades

Registry and factory live behind proxies. `INFRA_UPGRADE_ROLE` (roleId `6`) is authorized to upgrade them via the associated ProxyAdmin. Bridge encoders are also upgradeable under the same role.

## Deployment dependencies

```
MakinaLiteRegistry  ←─────────────┐
   ▲                              │
   │ holds                        │ reads from
   │                              │
ModuleFactory ──── deploys ───►  MakinaLiteModule (clone)
                                  │
                                  ├── reads bridgeEncoders, feeCollector, flashLoanModule
                                  │   via registry on each call
                                  │
FlashLoanModule (immutable refs)  │
   ├── moduleFactory  ←────── used to validate `taker`
   └── morpho
```
