# Core: Factories

Contract deployment factories for the Makina protocol.

## IHubCoreFactory

Factory for deploying Machine and PreDepositVault instances on the hub chain.

```solidity
interface IHubCoreFactory is IBridgeAdapterFactory {
    event MachineCreated(address indexed machine, address indexed shareToken);
    event PreDepositVaultCreated(address indexed preDepositVault, address indexed shareToken);
    event ShareTokenCreated(address indexed shareToken);

    function isPreDepositVault(address preDepositVault) external view returns (bool);
    function isMachine(address machine) external view returns (bool);

    /// @notice Deploy a PreDepositVault for collecting deposits before Machine launch
    function createPreDepositVault(
        IPreDepositVault.PreDepositVaultInitParams calldata params,
        address depositToken,
        address accountingToken,
        string memory tokenName,
        string memory tokenSymbol
    ) external returns (address preDepositVault);

    /// @notice Deploy Machine and migrate from existing PreDepositVault
    function createMachineFromPreDeposit(
        IMachine.MachineInitParams calldata mParams,
        ICaliber.CaliberInitParams calldata cParams,
        IMakinaGovernable.MakinaGovernableInitParams calldata mgParams,
        address preDepositVault,
        bytes32 salt
    ) external returns (address machine);

    /// @notice Deploy a new Machine directly
    function createMachine(
        IMachine.MachineInitParams calldata mParams,
        ICaliber.CaliberInitParams calldata cParams,
        IMakinaGovernable.MakinaGovernableInitParams calldata mgParams,
        address accountingToken,
        string memory tokenName,
        string memory tokenSymbol,
        bytes32 salt
    ) external returns (address machine);
}
```

## ISpokeCoreFactory

Factory for deploying Caliber instances on spoke chains.

```solidity
interface ISpokeCoreFactory is IBridgeAdapterFactory {
    event CaliberMailboxCreated(address indexed mailbox, address indexed caliber, address indexed hubMachine);

    function isCaliberMailbox(address mailbox) external view returns (bool);

    /// @notice Deploy a new Caliber on a spoke chain
    function createCaliber(
        ICaliber.CaliberInitParams calldata cParams,
        IMakinaGovernable.MakinaGovernableInitParams calldata mgParams,
        address accountingToken,
        address hubMachine,
        bytes32 salt
    ) external returns (address caliber);
}
```

## IBridgeAdapterFactory

Base factory interface for bridge adapter deployment.

```solidity
interface IBridgeAdapterFactory {
    // Inherited by IHubCoreFactory and ISpokeCoreFactory
    // Bridge adapter creation is handled through IBridgeController
}
```

## Deployment Flow

### 1. PreDepositVault Phase
```solidity
// Deploy PreDepositVault to collect early deposits
address pdv = IHubCoreFactory(factory).createPreDepositVault(
    IPreDepositVault.PreDepositVaultInitParams({
        initialShareLimit: 1000000e18,
        initialWhitelistMode: true,
        initialRiskManager: riskManager,
        initialAuthority: authority
    }),
    USDC,      // depositToken
    USDC,      // accountingToken
    "Makina Vault Shares",
    "mVault"
);
```

### 2. Machine Launch
```solidity
// Migrate PreDepositVault to Machine
address machine = IHubCoreFactory(factory).createMachineFromPreDeposit(
    IMachine.MachineInitParams({
        initialDepositor: depositor,
        initialRedeemer: redeemer,
        initialFeeManager: feeManager,
        initialCaliberStaleThreshold: 1 days,
        initialMaxFixedFeeAccrualRate: 0.02e18, // 2% per year
        initialMaxPerfFeeAccrualRate: 0.20e18, // 20%
        initialFeeMintCooldown: 1 hours,
        initialShareLimit: type(uint256).max
    }),
    ICaliber.CaliberInitParams({
        initialPositionStaleThreshold: 1 days,
        initialAllowedInstrRoot: merkleRoot,
        initialTimelockDuration: 2 days,
        initialMaxPositionIncreaseLossBps: 100, // 1%
        initialMaxPositionDecreaseLossBps: 100,
        initialMaxSwapLossBps: 50, // 0.5%
        initialCooldownDuration: 1 hours
    }),
    IMakinaGovernable.MakinaGovernableInitParams({
        initialMechanic: mechanic,
        initialSecurityCouncil: council,
        initialRiskManager: riskManager,
        initialRiskManagerTimelock: timelock,
        initialAuthority: authority
    }),
    pdv,
    salt
);
```

### 3. Spoke Caliber Deployment
```solidity
// On spoke chain (e.g., Arbitrum)
address caliber = ISpokeCoreFactory(spokeFactory).createCaliber(
    caliberParams,
    governableParams,
    USDC,           // accountingToken on spoke
    hubMachine,     // Machine address on hub
    salt
);
```
