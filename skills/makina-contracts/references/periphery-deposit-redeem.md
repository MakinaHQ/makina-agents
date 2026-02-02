# Periphery: Depositors & Redeemers

User deposit and redemption flow contracts.

## IDirectDepositor

Simple depositor for direct Machine deposits.

```solidity
interface IDirectDepositor is IMachinePeriphery {
    /// @notice Deposit accounting tokens and mint shares
    /// @param assets Amount of accounting tokens to deposit
    /// @param receiver Address to receive minted shares
    /// @param minShares Minimum shares to mint (slippage protection)
    /// @param referralKey Optional referral tracking identifier
    /// @return shares Amount of shares minted
    function deposit(uint256 assets, address receiver, uint256 minShares, bytes32 referralKey)
        external
        returns (uint256);
}
```

### Usage
```solidity
// Approve tokens first
IERC20(accountingToken).approve(depositor, assets);

// Deposit
uint256 shares = IDirectDepositor(depositor).deposit(
    1000e6,                     // 1000 USDC
    msg.sender,                 // receiver
    990e18,                     // min 990 shares
    bytes32(0)                  // no referral
);
```

## IAsyncRedeemer

Asynchronous redemption with finalization delay.

```solidity
interface IAsyncRedeemer is IMachinePeriphery {
    event RedeemRequestCreated(uint256 indexed requestId, uint256 shares, address indexed receiver);
    event RedeemRequestClaimed(uint256 indexed requestId, uint256 shares, uint256 assets, address indexed receiver);
    event RedeemRequestsFinalized(uint256 indexed fromRequestId, uint256 indexed toRequestId, uint256 totalShares, uint256 totalAssets);
    event FinalizationDelayChanged(uint256 indexed oldDelay, uint256 indexed newDelay);
    event MinRedeemAmountChanged(uint256 indexed oldMinRedeemAmount, uint256 indexed newMinRedeemAmount);

    struct RedeemRequest {
        uint256 shares;
        uint256 assets;
        uint256 requestTime;
    }

    // View functions
    function nextRequestId() external view returns (uint256);
    function lastFinalizedRequestId() external view returns (uint256);
    function finalizationDelay() external view returns (uint256);
    function minRedeemAmount() external view returns (uint256);
    function getShares(uint256 requestId) external view returns (uint256);
    function getClaimableAssets(uint256 requestId) external view returns (uint256);
    function previewFinalizeRequests(uint256 upToRequestId) external view returns (uint256, uint256);

    // State-changing functions
    function requestRedeem(uint256 shares, address receiver, uint256 minAssets) external returns (uint256);
    function finalizeRequests(uint256 upToRequestId, uint256 minAssets) external returns (uint256, uint256);
    function claimAssets(uint256 requestId) external returns (uint256);
    function setFinalizationDelay(uint256 newDelay) external;
    function setMinRedeemAmount(uint256 newMinRedeemAmount) external;
}
```

### Async Redemption Flow

```solidity
// Step 1: Request redemption (user)
// Approve shares first
IERC20(shareToken).approve(redeemer, shares);

uint256 requestId = IAsyncRedeemer(redeemer).requestRedeem(
    1000e18,        // shares to redeem
    msg.sender,     // receiver
    990e6           // min assets expected
);

// Step 2: Wait for finalization delay to pass

// Step 3: Finalize requests (anyone can call)
(uint256 totalShares, uint256 totalAssets) = IAsyncRedeemer(redeemer).finalizeRequests(
    requestId,      // finalize up to this request
    minAssets       // minimum total assets to receive
);

// Step 4: Claim assets (user)
uint256 assets = IAsyncRedeemer(redeemer).claimAssets(requestId);
```

### Why Async?
- Allows protocol to orderly unwind positions
- Prevents bank-run scenarios
- Ensures fair redemption pricing
- Finalization delay typically 24-72 hours

## IMachinePeriphery

Base interface for periphery contracts linked to a Machine.

```solidity
interface IMachinePeriphery {
    event MachineSet(address indexed machine);

    function initialize(bytes calldata _data) external;
    function machine() external view returns (address);
    function setMachine(address _machine) external;
}
```

## Typical User Flow

```
1. User approves accounting token to DirectDepositor
       ↓
2. User calls deposit() on DirectDepositor
       ↓
3. DirectDepositor deposits to Machine
       ↓
4. User receives MachineShare tokens
       ↓
   ... time passes, value accrues ...
       ↓
5. User approves shares to AsyncRedeemer
       ↓
6. User calls requestRedeem()
       ↓
7. Request enters queue
       ↓
   ... finalization delay passes ...
       ↓
8. Anyone calls finalizeRequests()
       ↓
9. User calls claimAssets() to receive tokens
```
