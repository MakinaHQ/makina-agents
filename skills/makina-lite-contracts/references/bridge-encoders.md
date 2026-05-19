# Bridge Encoders

Per-protocol contracts that translate a generic `BridgeOrder` into the (`approvalTarget`, `executionTarget`, `value`, `calldata`) tuple consumed by `BridgeComponent`. Encoders are `AccessManagedUpgradeable` and looked up by `bridgeId` in the `MakinaLiteRegistry`.

## IBridgeEncoder (common base)

```solidity
interface IBridgeEncoder {
    function getBridgeTransferData(IBridgeComponent.BridgeOrder calldata order, bool lockdownMode)
        external
        view
        returns (address approvalTarget, address executionTarget, uint256 value, bytes memory cd);
}
```

`BridgeOrder.extraData` is bridge-specific (see each section). When `approvalTarget == address(0)`, no approval is needed.

## IAcrossV4BridgeEncoder

Routes tokens through the Across V4 SpokePool using `depositV3Now`.

```solidity
interface IAcrossV4BridgeEncoder is IBridgeEncoder {
    event RouteAdded(address indexed inputToken, uint256 indexed foreignChainId, address indexed outputToken);
    event RouteRemoved(address indexed inputToken, uint256 indexed foreignChainId, address indexed outputToken);

    function acrossV4SpokePool() external view returns (address);
    function isRouteRegistered(address inputToken, uint256 foreignChainId, address outputToken) external view returns (bool);

    function addRoute(address inputToken, uint256 foreignChainId, address outputToken) external;    // INFRA_CONFIG_ROLE
    function removeRoute(address inputToken, uint256 foreignChainId, address outputToken) external; // INFRA_CONFIG_ROLE
}
```

- `extraData` encoding: `abi.encode(address outputToken, address refundAddress, uint32 fillDeadlineOffset)`.
- `refundAddress` must be non-zero.
- In lockdown mode the `(inputToken, destinationChainId, outputToken)` route must be registered. Outside lockdown, any output token is allowed.
- Encodes `IAcrossV4SpokePool.depositV3Now` with `exclusiveRelayer = address(0)`, `exclusivityDeadline = 0`, empty `message`.
- Returns `(spokePool, spokePool, 0, calldata)` — single approval and execution target, no native value.

## ICctpV2BridgeEncoder

Bridges tokens through Circle's CCTP V2 using `depositForBurnWithHook`. Maintains a bidirectional mapping between EVM chain IDs and CCTP domains.

```solidity
interface ICctpV2BridgeEncoder is IBridgeEncoder {
    event CctpDomainRegistered(uint256 indexed evmChainId, uint32 indexed cctpDomain);

    function cctpV2TokenMessenger() external view returns (address);
    function getCctpDomain(uint256 evmChainId) external view returns (uint32);

    function setCctpDomain(uint256 evmChainId, uint32 cctpDomain) external; // INFRA_CONFIG_ROLE
}
```

- `extraData` encoding: `abi.encode(uint32 minFinalityThreshold)`.
- Hardcoded: chainId `1` ↔ CCTP domain `0` (Ethereum mainnet); cannot be overwritten (`ProtectedChainId` / `ProtectedCctpDomain`).
- `minOutputAmount` must not exceed `inputAmount`; the difference becomes `maxFee` for the CCTP burn.
- Hook data is a fixed constant (`"cctp-forward\0\0\0…"`) used by Circle's forwarding hook.
- Returns `(tokenMessenger, tokenMessenger, 0, calldata)`.
- No extra registration check in lockdown mode beyond the chain ↔ domain mapping.

## ILayerZeroV2BridgeEncoder

Bridges tokens through LayerZero V2 OFT contracts. Maintains a mapping of EVM chain IDs to LayerZero endpoint IDs, and a registry of allowed OFTs.

```solidity
interface ILayerZeroV2BridgeEncoder is IBridgeEncoder {
    event LzEndpointIdRegistered(uint256 indexed evmChainId, uint32 indexed lzEndpointId);
    event OftAdded(address indexed oft);
    event OftRemoved(address indexed oft);

    function getLzEndpointId(uint256 evmChainId) external view returns (uint32);
    function isOftRegistered(address oft) external view returns (bool);

    function setLzEndpointId(uint256 evmChainId, uint32 lzEndpointId) external; // INFRA_CONFIG_ROLE
    function addOft(address oft) external;                                       // INFRA_CONFIG_ROLE
    function removeOft(address oft) external;                                    // INFRA_CONFIG_ROLE
}
```

- `extraData` encoding: `abi.encode(address oft, uint128 lzReceiveGas, uint256 maxValue)`.
- `IOFT(oft).token()` must equal `order.inputToken` (`OftMismatch`).
- In lockdown mode the OFT must be in `isOftRegistered`.
- Quotes are performed at encode time:
  - `quoteSend(...)`'s `nativeFee` must be ≤ `maxValue` (`ExceededMaxFee`).
  - `quoteOFT(...)` is checked for amount-sent / amount-received consistency.
- Returns:
  - `approvalTarget = oft` if `IOFT(oft).approvalRequired()`, else `address(0)`.
  - `executionTarget = oft`.
  - `value = mf.nativeFee` — the module must hold native to pay this fee (see `sweepNative`).
  - `calldata = IOFT.send(sendParam, mf, msg.sender)`.

`lzReceiveGas != 0` builds a v3 OFT options blob using the executor prefix `0x000301001101`; otherwise no extra options are passed.

## Bridge ID conventions

`bridgeId` is a `uint16` mapped to an encoder by `MakinaLiteRegistry.setBridgeEncoder`. The mapping is deployment-specific; consult the registry rather than hardcoding IDs.

## Summary: lockdown-mode enforcement per bridge

| Bridge | Recipient whitelist | Loss cap | Bridge-specific check |
|--------|---------------------|----------|------------------------|
| Across V4 | ✓ (recipient + chain) | ✓ | Route `(in, chain, out)` registered |
| CCTP V2 | ✓ | ✓ | EVM chain ID → CCTP domain mapped |
| LayerZero V2 | ✓ | ✓ | OFT in `isOftRegistered`, EVM chain ID → LZ EID mapped, native fee ≤ `maxValue` |

Loss cap and recipient whitelist are enforced by `BridgeComponent` itself; the bridge-specific check is enforced by the encoder via its `lockdownMode` flag.
