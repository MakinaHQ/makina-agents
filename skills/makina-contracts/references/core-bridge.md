# Core: Bridge

Cross-chain bridge adapters and controllers.

## IBridgeController

Controller interface for managing bridge adapters.

```solidity
interface IBridgeController {
    event BridgeAdapterCreated(uint16 indexed bridgeId, address indexed adapter);
    event MaxBridgeLossBpsChanged(uint16 indexed bridgeId, uint256 indexed oldMaxBridgeLossBps, uint256 indexed newMaxBridgeLossBps);
    event BridgingStateReset(address indexed token);
    event OutTransferEnabledSet(uint256 indexed bridgeId, bool enabled);

    function isBridgeSupported(uint16 bridgeId) external view returns (bool);
    function isOutTransferEnabled(uint16 bridgeId) external view returns (bool);
    function getBridgeAdapter(uint16 bridgeId) external view returns (address);
    function getMaxBridgeLossBps(uint16 bridgeId) external view returns (uint256);

    /// @notice Deploy a new bridge adapter
    function createBridgeAdapter(uint16 bridgeId, uint256 initialMaxBridgeLossBps, bytes calldata initData) external returns (address);

    /// @notice Set max allowed slippage for bridge transfers
    function setMaxBridgeLossBps(uint16 bridgeId, uint256 maxBridgeLossBps) external;

    /// @notice Enable/disable outgoing transfers for a bridge
    function setOutTransferEnabled(uint16 bridgeId, bool enabled) external;

    /// @notice Execute a scheduled outgoing bridge transfer
    function sendOutBridgeTransfer(uint16 bridgeId, uint256 transferId, bytes calldata data) external;

    /// @notice Authorize an incoming bridge message
    function authorizeInBridgeTransfer(uint16 bridgeId, bytes32 messageHash) external;

    /// @notice Claim received tokens from bridge adapter
    function claimInBridgeTransfer(uint16 bridgeId, uint256 transferId) external;

    /// @notice Cancel a pending outgoing transfer
    function cancelOutBridgeTransfer(uint16 bridgeId, uint256 transferId) external;

    /// @notice Reset bridge accounting state for a token
    function resetBridgingState(address token) external;
}
```

## IBridgeAdapter

Interface for individual bridge adapter implementations.

```solidity
interface IBridgeAdapter {
    event OutBridgeTransferScheduled(uint256 indexed transferId, bytes32 indexed messageHash);
    event OutBridgeTransferSent(uint256 indexed transferId);
    event OutBridgeTransferCancelled(uint256 indexed transferId);
    event InBridgeTransferAuthorized(bytes32 indexed messageHash);
    event InBridgeTransferReceived(uint256 indexed transferId);
    event InBridgeTransferClaimed(uint256 indexed transferId);
    event PendingFundsWithdrawn(address indexed token, uint256 amount);

    struct OutBridgeTransfer {
        address recipient;
        uint256 destinationChainId;
        address inputToken;
        uint256 inputAmount;
        address outputToken;
        uint256 minOutputAmount;
        bytes encodedMessage;
    }

    struct InBridgeTransfer {
        address sender;
        uint256 originChainId;
        address inputToken;
        uint256 inputAmount;
        address outputToken;
        uint256 outputAmount;
    }

    struct BridgeMessage {
        uint256 outTransferId;
        address sender;
        address recipient;
        uint256 originChainId;
        uint256 destinationChainId;
        address inputToken;
        uint256 inputAmount;
        address outputToken;
        uint256 minOutputAmount;
    }

    function controller() external view returns (address);
    function bridgeId() external view returns (uint16);
    function approvalTarget() external view returns (address);
    function executionTarget() external view returns (address);
    function receiveSource() external view returns (address);
    function nextOutTransferId() external view returns (uint256);
    function nextInTransferId() external view returns (uint256);

    function scheduleOutBridgeTransfer(
        uint256 destinationChainId,
        address recipient,
        address inputToken,
        uint256 inputAmount,
        address outputToken,
        uint256 minOutputAmount
    ) external;

    function sendOutBridgeTransfer(uint256 transferId, bytes calldata data) external;
    function outBridgeTransferCancelDefault(uint256 transferId) external view returns (uint256);
    function cancelOutBridgeTransfer(uint256 transferId) external;
    function authorizeInBridgeTransfer(bytes32 messageHash) external;
    function claimInBridgeTransfer(uint256 transferId) external;
    function withdrawPendingFunds(address token) external;
}
```

## IBridgeConfig

Configuration for supported bridge routes.

```solidity
interface IBridgeConfig {
    /// @notice Check if a bridge route is supported
    function isRouteSupported(address inputToken, uint256 foreignChainId, address outputToken) external view returns (bool);
}
```

## Supported Bridges

The protocol supports these bridge IDs:
- **1**: Across V3
- **2**: LayerZero V2

## Bridge Flow

### Outgoing Transfer (Hub -> Spoke)
```solidity
// 1. Schedule transfer
IBridgeAdapter(adapter).scheduleOutBridgeTransfer(
    arbitrumChainId,    // destinationChainId
    spokeCaliberMailbox,// recipient
    USDC,               // inputToken
    1000000e6,          // inputAmount
    USDC,               // outputToken
    990000e6            // minOutputAmount (with slippage)
);

// 2. Execute transfer
IBridgeController(machine).sendOutBridgeTransfer(
    bridgeId,
    transferId,
    bridgeSpecificData
);
```

### Incoming Transfer (Spoke -> Hub)
```solidity
// 1. Authorize the incoming message
IBridgeController(machine).authorizeInBridgeTransfer(bridgeId, messageHash);

// 2. Claim the tokens
IBridgeController(machine).claimInBridgeTransfer(bridgeId, transferId);
```

### High-level Transfer via Machine
```solidity
// Transfer tokens from Machine to Spoke Caliber
IMachine(machine).transferToSpokeCaliber(
    bridgeId,
    arbitrumChainId,
    USDC,
    1000000e6,
    990000e6  // minOutputAmount
);
```
