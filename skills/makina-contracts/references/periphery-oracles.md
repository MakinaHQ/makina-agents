# Periphery: Oracles

Share price oracle contracts for external integrations.

## IMachineShareOracle

Oracle providing MachineShare price for external protocols.

```solidity
interface IMachineShareOracle {
    error InvalidShareOwner();

    event ShareOwnerMigrated(address indexed oldShareOwner, address indexed newShareOwner);

    /// @notice Initialize the oracle
    /// @param _shareOwner Machine or PreDepositVault address
    /// @param _decimals Price decimals (e.g., 8 for Chainlink compatibility)
    function initialize(address _shareOwner, uint8 _decimals) external;

    /// @notice Oracle decimals
    function decimals() external view returns (uint8);

    /// @notice Oracle description string
    function description() external view returns (string memory);

    /// @notice Current share owner (Machine or PreDepositVault)
    function shareOwner() external view returns (address);

    /// @notice Get share price in accounting token terms
    /// @return sharePrice Price scaled to `decimals` precision
    function getSharePrice() external view returns (uint256 sharePrice);

    /// @notice Notify oracle of PreDepositVault migration to Machine
    function notifyPdvMigration() external;
}
```

## IMachineShareOracleFactory

Factory for deploying share price oracles.

```solidity
interface IMachineShareOracleFactory {
    event MachineShareOracleCreated(address indexed oracle);
    event MachineShareOracleBeaconChanged(address indexed oldBeacon, address indexed newBeacon);

    function machineShareOracleBeacon() external view returns (address);
    function isMachineShareOracle(address oracle) external view returns (bool);

    /// @notice Create oracle for a Machine or PreDepositVault
    /// @param shareOwner Address of Machine or PreDepositVault
    /// @param decimals Price decimals
    function createMachineShareOracle(address shareOwner, uint8 decimals) external returns (address);

    function setMachineShareOracleBeacon(address _machineShareOracleBeacon) external;
}
```

## Usage Examples

### Deploy Oracle
```solidity
address oracle = IMachineShareOracleFactory(factory).createMachineShareOracle(
    machine,    // or preDepositVault
    8           // 8 decimals for Chainlink compatibility
);
```

### Read Share Price
```solidity
uint256 price = IMachineShareOracle(oracle).getSharePrice();
// Returns price with 8 decimals
// e.g., 1.05 USDC per share = 105000000
```

### Integration with Lending Protocols

The oracle is designed for integration with:
- **Morpho**: Used to price MachineShare as collateral
- **Aave**: Can be used via adapter
- **Other protocols**: Chainlink-compatible interface

```solidity
// Example: Using with MetaMorpho
// The oracle provides the share price that Morpho uses
// to calculate collateral value

uint256 collateralValue = shareAmount * oracle.getSharePrice() / 10**oracle.decimals();
```

## IMetaMorphoOracleFactory

Factory for MetaMorpho-specific oracle integrations (if needed).

```solidity
interface IMetaMorphoOracleFactory {
    // Creates oracles specifically configured for MetaMorpho integration
}
```

## Price Calculation

The oracle calculates share price from the Machine:

```
sharePrice = Machine.convertToAssets(10^18) / 10^(18 - decimals)
```

This represents how many accounting tokens (e.g., USDC) one share is worth.

## PreDepositVault Migration

When a PreDepositVault migrates to a Machine:

```solidity
// Anyone can call after migration
IMachineShareOracle(oracle).notifyPdvMigration();
```

This updates the oracle to read from the Machine instead of the PreDepositVault, optimizing gas costs for future price queries.

## Oracle Security

- Oracle reads directly from Machine/PreDepositVault
- No external price feeds needed (self-pricing)
- Price manipulation resistance through:
  - Accounting staleness checks
  - Position value limits
  - Caliber staleness thresholds
