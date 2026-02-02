# Periphery: Fee Management

Fee calculation and distribution contracts.

## IWatermarkFeeManager

Fee manager using high-water mark for performance fees.

```solidity
interface IWatermarkFeeManager is IFeeManager, ISecurityModuleReference, IMachinePeriphery {
    // Events
    event MgmtFeeRatePerSecondChanged(uint256 oldRate, uint256 newRate);
    event SmFeeRatePerSecondChanged(uint256 oldRate, uint256 newRate);
    event PerfFeeRateChanged(uint256 oldRate, uint256 newRate);
    event MgmtFeeSplitChanged();
    event PerfFeeSplitChanged();
    event SecurityModuleSet(address indexed securityModule);
    event WatermarkReset(uint256 indexed newWatermark);

    struct WatermarkFeeManagerInitParams {
        uint256 initialMgmtFeeRatePerSecond;
        uint256 initialSmFeeRatePerSecond;
        uint256 initialPerfFeeRate;
        uint256[] initialMgmtFeeSplitBps;
        address[] initialMgmtFeeReceivers;
        uint256[] initialPerfFeeSplitBps;
        address[] initialPerfFeeReceivers;
    }

    // View functions
    function mgmtFeeRatePerSecond() external view returns (uint256);
    function smFeeRatePerSecond() external view returns (uint256);
    function perfFeeRate() external view returns (uint256);
    function mgmtFeeReceivers() external view returns (address[] memory);
    function mgmtFeeSplitBps() external view returns (uint256[] memory);
    function perfFeeReceivers() external view returns (address[] memory);
    function perfFeeSplitBps() external view returns (uint256[] memory);
    function sharePriceWatermark() external view returns (uint256);

    // State-changing functions
    function resetSharePriceWatermark(uint256 sharePrice) external;
    function setMgmtFeeRatePerSecond(uint256 newMgmtFeeRatePerSecond) external;
    function setSmFeeRatePerSecond(uint256 newSmFeeRatePerSecond) external;
    function setPerfFeeRate(uint256 newPerfFeeRate) external;
    function setMgmtFeeSplit(address[] calldata newMgmtFeeReceivers, uint256[] calldata newMgmtFeeSplitBps) external;
    function setPerfFeeSplit(address[] calldata newPerfFeeReceivers, uint256[] calldata newPerfFeeSplitBps) external;
}
```

## IFeeManager (Base Interface)

```solidity
interface IFeeManager {
    /// @notice Calculate fixed management fee
    /// @param shareSupply Total supply of shares
    /// @param elapsedTime Time since last fee calculation
    function calculateFixedFee(uint256 shareSupply, uint256 elapsedTime) external returns (uint256);

    /// @notice Calculate performance fee based on share price increase
    /// @param currentShareSupply Current share supply
    /// @param oldSharePrice Previous share price
    /// @param newSharePrice Current share price
    /// @param elapsedTime Time since last calculation
    function calculatePerformanceFee(
        uint256 currentShareSupply,
        uint256 oldSharePrice,
        uint256 newSharePrice,
        uint256 elapsedTime
    ) external returns (uint256);

    /// @notice Distribute calculated fees
    function distributeFees(uint256 fixedFee, uint256 perfFee) external;
}
```

## ISecurityModuleReference

Interface for contracts that reference a Security Module.

```solidity
interface ISecurityModuleReference {
    function securityModule() external view returns (address);
}
```

## Fee Types

### Management Fee (Fixed Fee)
- Accrues continuously over time
- Calculated as: `shareSupply * mgmtFeeRatePerSecond * elapsedTime`
- Typical rate: 1-2% annually

### Security Module Fee
- Additional fee for Security Module stakers
- Calculated similarly to management fee
- Incentivizes protocol insurance participation

### Performance Fee
- Only charged on new profits above high-water mark
- Calculated as: `profitShares * perfFeeRate`
- Typical rate: 10-20% of profits
- High-water mark prevents double-charging on recovered losses

## High-Water Mark

The watermark tracks the highest share price achieved:

```
Initial: watermark = 1.0
Day 10:  share price rises to 1.1 → watermark = 1.1, perf fee charged on 0.1 profit
Day 20:  share price drops to 1.05 → watermark = 1.1, no perf fee
Day 30:  share price rises to 1.15 → watermark = 1.15, perf fee charged on 0.05 profit
```

## Fee Distribution

Fees are distributed to multiple recipients based on configured splits:

```solidity
// Example configuration
mgmtFeeReceivers = [treasury, team, strategist]
mgmtFeeSplitBps = [5000, 3000, 2000]  // 50%, 30%, 20%

perfFeeReceivers = [treasury, strategist]
perfFeeSplitBps = [6000, 4000]  // 60%, 40%
```

## Fee Calculation Flow

```
1. Machine.updateTotalAum() is called
       ↓
2. New share price calculated
       ↓
3. FeeManager.calculateFixedFee() → management fees
       ↓
4. FeeManager.calculatePerformanceFee() → performance fees
       ↓
5. FeeManager.distributeFees() → mint new shares to recipients
       ↓
6. Total share supply increases (dilution = fee payment)
```

## Rate Conversion

Fee rates are stored per-second for precision:

```solidity
// Convert annual rate to per-second
// 2% annual = 0.02 / (365.25 * 24 * 60 * 60) = ~6.34e-10

uint256 annualRate = 0.02e18;  // 2%
uint256 ratePerSecond = annualRate / 31557600;  // seconds per year
```

## Reading Current Fees

```solidity
// Get fee configuration
uint256 mgmtRate = IWatermarkFeeManager(feeManager).mgmtFeeRatePerSecond();
uint256 perfRate = IWatermarkFeeManager(feeManager).perfFeeRate();
uint256 watermark = IWatermarkFeeManager(feeManager).sharePriceWatermark();

// Get fee recipients
address[] memory mgmtReceivers = IWatermarkFeeManager(feeManager).mgmtFeeReceivers();
uint256[] memory mgmtSplits = IWatermarkFeeManager(feeManager).mgmtFeeSplitBps();
```
