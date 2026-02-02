# Core: Swap & Utilities

Swap module and utility interfaces.

## ISwapModule

Token swap functionality for Calibers.

```solidity
interface ISwapModule {
    event Swap(
        address indexed sender,
        uint16 swapperId,
        address indexed inputToken,
        address indexed outputToken,
        uint256 inputAmount,
        uint256 outputAmount
    );
    event SwapperTargetsSet(uint16 indexed swapper, address approvalTarget, address executionTarget);

    struct SwapperTargets {
        address approvalTarget;
        address executionTarget;
    }

    struct SwapOrder {
        uint16 swapperId;
        bytes data;
        address inputToken;
        address outputToken;
        uint256 inputAmount;
        uint256 minOutputAmount;
    }

    function getSwapperTargets(uint16 swapperId) external view returns (address approvalTarget, address executionTarget);
    function swap(SwapOrder calldata order) external returns (uint256);
    function setSwapperTargets(uint16 swapperId, address approvalTarget, address executionTarget) external;
}
```

### Usage Example
```solidity
// Execute a swap through the Caliber
ICaliber(caliber).swap(
    ISwapModule.SwapOrder({
        swapperId: 1,           // e.g., 1inch
        data: swapCalldata,     // encoded swap data
        inputToken: WETH,
        outputToken: USDC,
        inputAmount: 1e18,
        minOutputAmount: 3000e6 // minimum USDC out
    })
);
```

## IFeeManager

Fee calculation interface (implemented by WatermarkFeeManager in periphery).

```solidity
interface IFeeManager {
    /// @notice Calculate fixed management fee
    function calculateFixedFee(uint256 shareSupply, uint256 elapsedTime) external returns (uint256);

    /// @notice Calculate performance fee based on share price increase
    function calculatePerformanceFee(
        uint256 currentShareSupply,
        uint256 oldSharePrice,
        uint256 newSharePrice,
        uint256 elapsedTime
    ) external returns (uint256);

    /// @notice Distribute fees to recipients
    function distributeFees(uint256 fixedFee, uint256 perfFee) external;
}
```

## IFlashloanAggregator

Aggregator for accessing flashloans from multiple providers.

```solidity
interface IFlashloanAggregator {
    enum FlashloanProvider {
        AAVE_V3,
        BALANCER_V2,
        BALANCER_V3,
        MORPHO,
        DSS_FLASH
    }

    struct FlashloanRequest {
        FlashloanProvider provider;
        ICaliber.Instruction instruction;
        address token;
        uint256 amount;
    }

    function requestFlashloan(FlashloanRequest calldata request) external;
}
```

### Flashloan Usage
```solidity
// Execute flashloan-based position management
IFlashloanAggregator(aggregator).requestFlashloan(
    IFlashloanAggregator.FlashloanRequest({
        provider: IFlashloanAggregator.FlashloanProvider.AAVE_V3,
        instruction: managementInstruction,
        token: USDC,
        amount: 1000000e6
    })
);
```

## IWhitelist

Generic whitelist interface.

```solidity
interface IWhitelist {
    event UserWhitelistingChanged(address indexed user, bool indexed whitelisted);
    event WhitelistStatusChanged(bool indexed enabled);

    function isWhitelistEnabled() external view returns (bool);
    function isWhitelistedUser(address user) external view returns (bool);
    function setWhitelistStatus(bool enabled) external;
    function setWhitelistedUsers(address[] calldata users, bool whitelisted) external;
}
```

## Common Swapper IDs

| ID | Protocol |
|----|----------|
| 1 | 1inch |
| 2 | 0x |
| 3 | Paraswap |
| 4 | Cowswap |

## Weiroll Integration

Calibers use Weiroll for executing complex multi-step instructions. The `IWeirollVM` interface is used internally:

```solidity
interface IWeirollVM {
    // Weiroll virtual machine for executing instruction sequences
    // Commands and state are encoded in Instruction struct
}
```

Instructions use a Merkle tree for authorization, allowing the DAO to approve specific operations while maintaining security.
