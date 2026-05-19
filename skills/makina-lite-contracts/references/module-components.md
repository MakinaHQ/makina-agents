# Module Components

`MakinaLiteModule` inherits four mix-in components. This page documents their interfaces and the lockdown-mode safety rules.

## IWeirollComponent

Position management via the [Weiroll](https://github.com/EnsoBuild/enso-weiroll) command-chaining framework executed through the Safe via `delegatecall`.

```solidity
interface IWeirollComponent {
    event AllowedInstrRootChanged(bytes32 indexed oldRoot, bytes32 indexed newRoot);
    event AccountingCurrencyChanged(address indexed oldAccountingCurrency, address indexed newAccountingCurrency);
    event MaxPositionIncreaseLossBpsChanged(uint256 oldMaxPositionIncreaseLossBps, uint256 newMaxPositionIncreaseLossBps);
    event MaxPositionDecreaseLossBpsChanged(uint256 oldMaxPositionDecreaseLossBps, uint256 newMaxPositionDecreaseLossBps);
    event PositionManaged(bool indexed withValuation, bool indexed lockdownMode, uint256 indexed positionId, uint256 value);

    enum InstructionType {
        MANAGEMENT,
        ACCOUNTING,
        HARVEST,
        FLASHLOAN_MANAGEMENT
    }

    struct Instruction {
        uint256 positionId;
        bool isDebt;
        uint256 groupId;          // Reserved; only used for Core compatibility
        InstructionType instructionType;
        address[] affectedTokens;
        address[] positionTokens;
        bytes32[] commands;
        bytes[] state;
        uint128 stateBitmap;      // 1 bit per state slot: 1 = fixed in Merkle leaf, 0 = variable
        bytes32[] merkleProof;
    }

    function weirollVm() external view returns (address);
    function allowedInstrRoot() external view returns (bytes32);
    function accountingCurrency() external view returns (address); // address(0) => OracleRegistry reference currency
    function maxPositionIncreaseLossBps() external view returns (uint256);
    function maxPositionDecreaseLossBps() external view returns (uint256);

    function accountForPosition(Instruction calldata instruction) external returns (uint256 value);
    function accountForPositionBatch(Instruction[] calldata instructions, uint256[] calldata groupIds)
        external returns (uint256[] memory values);
    function managePosition(Instruction calldata mgmtInstruction, Instruction calldata acctInstruction)
        external returns (uint256 value, int256 change);
    function managePositionBatch(Instruction[] calldata mgmtInstructions, Instruction[] calldata acctInstructions)
        external returns (uint256[] memory values, int256[] memory changes);
    function manageFlashLoan(Instruction calldata instruction, address token, uint256 amount) external;
    function harvest(Instruction calldata instruction, ISwapComponent.SwapOrder[] calldata swapOrders) external;

    function setAllowedInstrRoot(bytes32 newAllowedInstrRoot) external;
    function setAccountingCurrency(address newAccountingCurrency) external;
    function setMaxPositionIncreaseLossBps(uint256 newMaxPositionIncreaseLossBps) external;
    function setMaxPositionDecreaseLossBps(uint256 newMaxPositionDecreaseLossBps) external;
}
```

### Instruction assumptions

Required for correctness — many are only enforced in lockdown mode but remain best practice always:

- **ACCOUNTING**: must not mutate state or balances; output resistant to manipulation; `affectedTokens` is exactly the tokens used to express the position size; output state starts with one amount per slot in `affectedTokens` order, followed by an end-of-args flag.
- **MANAGEMENT**: `affectedTokens` includes exactly the tokens spent; must not leave persistent ERC20 approvals from the Safe.
- **HARVEST**: receive-only operations; must not spend tokens initially held by the Safe.
- **FLASHLOAN_MANAGEMENT**: only changes balances of tokens in the outer `MANAGEMENT.affectedTokens`; must not leave persistent approvals; only callable inside a flash-loan callback by the `FlashLoanModule`.

### Lockdown mode validation matrix

In lockdown mode, `managePosition` requires `acctInstruction` and applies these checks based on the Safe's aggregate value change for `mgmtInstruction.affectedTokens`, whether the position is debt, and the sign of the position value delta:

```
┌──────────────────────┬───────────────┬──────────────────────┬───────────────────────────┐
│ Affected Tokens flow │ Debt Position │ Position Δ direction │ Action                    │
├──────────────────────┼───────────────┼──────────────────────┼───────────────────────────┤
│ Outflow              │ No            │ Decrease             │ Revert: Invalid direction │
│ Outflow              │ Yes           │ Increase             │ Revert: Invalid direction │
│ Outflow              │ No            │ Increase / Null      │ Minimum Δ Check           │
│ Outflow              │ Yes           │ Decrease / Null      │ Minimum Δ Check           │
│ Inflow / Null        │ No            │ Decrease             │ Maximum Δ Check           │
│ Inflow / Null        │ Yes           │ Increase             │ Maximum Δ Check           │
│ Inflow / Null        │ No            │ Increase / Null      │ No check (favorable move) │
│ Inflow / Null        │ Yes           │ Decrease / Null      │ No check (favorable move) │
└──────────────────────┴───────────────┴──────────────────────┴───────────────────────────┘
```

- **Minimum Δ Check**: position value gained must be within `maxPositionIncreaseLossBps` of tokens spent.
- **Maximum Δ Check**: tokens received must be within `maxPositionDecreaseLossBps` of position value lost.

`groupIds` in `accountForPositionBatch` is unused; it exists to preserve interface compatibility with Makina Core.

## ISwapComponent

Swaps via configurable DEX-aggregator targets, with a provider-set fee paid to the registry's `feeCollector`.

```solidity
interface ISwapComponent {
    event MaxSwapLossBpsChanged(uint256 oldMaxSwapLossBps, uint256 newMaxSwapLossBps);
    event Swap(uint16 indexed swapperId, address indexed inputToken, address indexed outputToken, uint256 inputAmount, uint256 outputAmount);
    event SwapFeeRateChanged(uint256 oldSwapFeeRate, uint256 newSwapFeeRate);
    event SwapperTargetsSet(uint16 indexed swapperId, address approvalTarget, address executionTarget);

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

    function maxSwapLossBps() external view returns (uint256);
    function swapFeeRate() external view returns (uint256); // 1e18 = 100%
    function getSwapperTargets(uint16 swapperId) external view returns (address approvalTarget, address executionTarget);

    function swap(SwapOrder calldata order) external;
    function setMaxSwapLossBps(uint256 newMaxSwapLossBps) external;       // Safe
    function setSwapFeeRate(uint256 newSwapFeeRate) external;             // Provider
    function setSwapperTargets(uint16 swapperId, address approvalTarget, address executionTarget) external; // Safe
}
```

Execution: pull `inputToken` from Safe → approve `approvalTarget` for `inputAmount` → call `executionTarget` with `data` → revoke approval → take `swapFeeRate` of output to `feeCollector` → forward remainder to Safe.

In lockdown mode the output value must be within `maxSwapLossBps` of the input value, both priced via `OracleRegistry`.

## IBridgeComponent

Outgoing cross-chain transfers; the module pulls the input token from the Safe and delegates encoding to a per-bridge `IBridgeEncoder` resolved through `MakinaLiteRegistry.getBridgeEncoder(bridgeId)`.

```solidity
interface IBridgeComponent {
    event BridgeTransferRecipientAdded(uint256 indexed foreignChainId, address indexed recipient);
    event BridgeTransferRecipientRemoved(uint256 indexed foreignChainId, address indexed recipient);
    event MaxBridgeLossBpsChanged(uint16 indexed bridgeId, uint256 indexed oldMaxBridgeLossBps, uint256 indexed newMaxBridgeLossBps);

    struct BridgeOrder {
        uint16 bridgeId;
        uint256 destinationChainId;
        address recipient;
        address inputToken;
        uint256 inputAmount;
        uint256 minOutputAmount;
        bytes extraData;
    }

    function getMaxBridgeLossBps(uint16 bridgeId) external view returns (uint256);
    function isWhitelistedRecipient(uint256 foreignChainId, address recipient) external view returns (bool);

    function sendOutBridgeTransfer(BridgeOrder calldata order) external;       // Operator
    function setMaxBridgeLossBps(uint16 bridgeId, uint256 maxBridgeLossBps) external; // Safe
    function addRecipient(uint256 foreignChainId, address recipient) external;        // Safe
    function removeRecipient(uint256 foreignChainId, address recipient) external;     // Safe
}
```

Lockdown-mode bridge transfers enforce:
- Recipient must be whitelisted for `destinationChainId`.
- `minOutputAmount / inputAmount` within `maxBridgeLossBps`.
- Bridge-specific registration: Across V4 route must be registered; LayerZero V2 OFT must be registered. (CCTP V2 has no extra registration check beyond domain mapping.)

See [bridge-encoders](bridge-encoders.md) for per-protocol details.

## IOracleRegistry

Aggregates Chainlink-compatible feeds (`AggregatorV2V3Interface`) into single-feed or two-feed routes. Prices in the reference currency (effectively USD) when `accountingCurrency == address(0)`, or in a chosen quote token using cross-token pricing.

```solidity
interface IOracleRegistry {
    event FeedRouteCleared(address indexed token);
    event FeedRouteRegistered(address indexed token, address indexed feed1, address indexed feed2);
    event FeedStaleThresholdChanged(address indexed feed, uint256 oldThreshold, uint256 newThreshold);

    struct FeedRoute {
        address feed1;
        address feed2;
    }

    function getFeedStaleThreshold(address feed) external view returns (uint256);
    function isFeedRouteRegistered(address token) external view returns (bool);
    function getFeedRoute(address token) external view returns (address feed1, address feed2);

    function getReferencePrice(address baseToken) external view returns (uint256); // 18 decimals
    function getPrice(address baseToken, address quoteToken) external view returns (uint256); // quoteToken decimals

    function setFeedRoute(
        address token,
        address feed1,
        uint256 stalenessThreshold1,
        address feed2,                       // address(0) for single-feed route
        uint256 stalenessThreshold2          // ignored if feed2 is address(0)
    ) external;
    function clearFeedRoute(address token) external;
    function setFeedStaleThreshold(address feed, uint256 threshold) external;
}
```

Two-feed routes combine `base → intermediate` and `intermediate → reference`. `getPrice(base, quote)` uses each token's reference-currency price to derive the cross rate. Staleness thresholds are per-feed (a feed may be shared across routes) — clearing a route preserves its feeds' staleness configs.
