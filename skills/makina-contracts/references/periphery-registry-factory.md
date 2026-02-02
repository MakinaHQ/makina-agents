# Periphery: Registry & Factory

Periphery deployment infrastructure.

## IHubPeripheryRegistry

Registry for periphery contract beacons and configuration.

```solidity
interface IHubPeripheryRegistry {
    event PeripheryFactoryChanged(address indexed oldPeripheryFactory, address indexed newPeripheryFactory);
    event DepositorBeaconChanged(uint16 indexed implemId, address indexed oldDepositorBeacon, address indexed newDepositorBeacon);
    event RedeemerBeaconChanged(uint16 indexed implemId, address indexed oldRedeemerBeacon, address indexed newRedeemerBeacon);
    event FeeManagerBeaconChanged(uint16 indexed implemId, address indexed oldFeeManagerBeacon, address indexed newFeeManagerBeacon);
    event SecurityModuleBeaconChanged(address indexed oldSecurityModuleBeacon, address indexed newSecurityModuleBeacon);

    function peripheryFactory() external view returns (address);
    function depositorBeacon(uint16 implemId) external view returns (address);
    function redeemerBeacon(uint16 implemId) external view returns (address);
    function feeManagerBeacon(uint16 implemId) external view returns (address);
    function securityModuleBeacon() external view returns (address);

    function setPeripheryFactory(address _peripheryFactory) external;
    function setDepositorBeacon(uint16 implemId, address _depositorBeacon) external;
    function setRedeemerBeacon(uint16 implemId, address _redeemerBeacon) external;
    function setFeeManagerBeacon(uint16 implemId, address _feeManagerBeacon) external;
    function setSecurityModuleBeacon(address _securityModuleBeacon) external;
}
```

## IHubPeripheryFactory

Factory for deploying periphery contracts.

```solidity
interface IHubPeripheryFactory {
    event DepositorCreated(address indexed depositor, uint16 indexed implemId);
    event RedeemerCreated(address indexed redeemer, uint16 indexed implemId);
    event FeeManagerCreated(address indexed feeManager, uint16 indexed implemId);
    event SecurityModuleCreated(address indexed securityModule);

    // Verification functions
    function isDepositor(address depositor) external view returns (bool);
    function isRedeemer(address redeemer) external view returns (bool);
    function isFeeManager(address feeManager) external view returns (bool);
    function isSecurityModule(address securityModule) external view returns (bool);

    // Implementation ID lookup
    function depositorImplemId(address depositor) external view returns (uint16);
    function redeemerImplemId(address redeemer) external view returns (uint16);
    function feeManagerImplemId(address feeManager) external view returns (uint16);

    // Configuration functions
    function setMachine(address machinePeriphery, address machine) external;
    function setSecurityModule(address feeManager, address securityModule) external;

    // Creation functions
    function createDepositor(uint16 implemId, bytes calldata initializationData) external returns (address depositor);
    function createRedeemer(uint16 implemId, bytes calldata initializationData) external returns (address redeemer);
    function createFeeManager(uint16 implemId, bytes calldata initializationData) external returns (address feeManager);
    function createSecurityModule(ISecurityModule.SecurityModuleInitParams calldata smParams) external returns (address securityModule);
}
```

## Implementation IDs

Different implementations of periphery contracts are identified by `implemId`:

| Component | implemId | Implementation |
|-----------|----------|----------------|
| Depositor | 1 | DirectDepositor |
| Redeemer | 1 | AsyncRedeemer |
| FeeManager | 1 | WatermarkFeeManager |

## Deployment Flow

### 1. Deploy Periphery Contracts

```solidity
// Create depositor
address depositor = IHubPeripheryFactory(factory).createDepositor(
    1,  // DirectDepositor implementation
    abi.encode(/* init params */)
);

// Create redeemer
address redeemer = IHubPeripheryFactory(factory).createRedeemer(
    1,  // AsyncRedeemer implementation
    abi.encode(finalizationDelay, minRedeemAmount)
);

// Create fee manager
address feeManager = IHubPeripheryFactory(factory).createFeeManager(
    1,  // WatermarkFeeManager implementation
    abi.encode(feeManagerInitParams)
);

// Create security module
address securityModule = IHubPeripheryFactory(factory).createSecurityModule(
    ISecurityModule.SecurityModuleInitParams({
        machineShare: machineShareToken,
        initialCooldownDuration: 7 days,
        initialMaxSlashableBps: 3000,  // 30%
        initialMinBalanceAfterSlash: 1e18
    })
);
```

### 2. Link to Machine

```solidity
// Set machine on periphery contracts
IHubPeripheryFactory(factory).setMachine(depositor, machine);
IHubPeripheryFactory(factory).setMachine(redeemer, machine);
IHubPeripheryFactory(factory).setMachine(feeManager, machine);
IHubPeripheryFactory(factory).setMachine(securityModule, machine);

// Link security module to fee manager
IHubPeripheryFactory(factory).setSecurityModule(feeManager, securityModule);
```

### 3. Configure Machine

```solidity
// Set periphery contracts on machine
IMachine(machine).setDepositor(depositor);
IMachine(machine).setRedeemer(redeemer);
IMachine(machine).setFeeManager(feeManager);
```

## Beacon Pattern

Periphery contracts use the beacon proxy pattern:
- Each implementation has a beacon contract
- Beacons can be upgraded to update all proxies
- Registry stores beacon addresses per implementation ID

```
Registry
    │
    ├── depositorBeacon[1] → DirectDepositorBeacon
    │                              │
    │                              └── DirectDepositor implementation
    │
    ├── redeemerBeacon[1] → AsyncRedeemerBeacon
    │                              │
    │                              └── AsyncRedeemer implementation
    │
    └── feeManagerBeacon[1] → WatermarkFeeManagerBeacon
                                   │
                                   └── WatermarkFeeManager implementation
```

## Verifying Contracts

```solidity
// Check if address is a valid periphery contract
bool isValidDepositor = IHubPeripheryFactory(factory).isDepositor(depositor);
bool isValidRedeemer = IHubPeripheryFactory(factory).isRedeemer(redeemer);
bool isValidFeeManager = IHubPeripheryFactory(factory).isFeeManager(feeManager);
bool isValidSM = IHubPeripheryFactory(factory).isSecurityModule(securityModule);

// Get implementation type
uint16 depositorType = IHubPeripheryFactory(factory).depositorImplemId(depositor);
```
