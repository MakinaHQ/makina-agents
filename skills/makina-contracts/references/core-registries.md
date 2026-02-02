# Core: Registries

Configuration registries for the Makina protocol.

## ICoreRegistry

Base registry interface for core protocol configuration.

```solidity
interface ICoreRegistry {
    event CoreFactoryChanged(address indexed oldCoreFactory, address indexed newCoreFactory);
    event OracleRegistryChanged(address indexed oldOracleRegistry, address indexed newOracleRegistry);
    event TokenRegistryChanged(address indexed oldTokenRegistry, address indexed newTokenRegistry);
    event SwapModuleChanged(address indexed oldSwapModule, address indexed newSwapModule);
    event FlashLoanModuleChanged(address indexed oldFlashLoanModule, address indexed newFlashLoanModule);
    event CaliberBeaconChanged(address indexed oldCaliberBeacon, address indexed newCaliberBeacon);
    event BridgeAdapterBeaconChanged(uint256 indexed bridgeId, address indexed oldBridgeAdapterBeacon, address indexed newBridgeAdapterBeacon);
    event BridgeConfigChanged(uint256 indexed bridgeId, address indexed oldBridgeConfig, address indexed newBridgeConfig);

    function coreFactory() external view returns (address);
    function oracleRegistry() external view returns (address);
    function tokenRegistry() external view returns (address);
    function swapModule() external view returns (address);
    function flashLoanModule() external view returns (address);
    function caliberBeacon() external view returns (address);
    function bridgeAdapterBeacon(uint16 bridgeId) external view returns (address);
    function bridgeConfig(uint16 bridgeId) external view returns (address);

    function setCoreFactory(address _coreFactory) external;
    function setOracleRegistry(address _oracleRegistry) external;
    function setTokenRegistry(address _tokenRegistry) external;
    function setSwapModule(address _swapModule) external;
    function setFlashLoanModule(address _flashLoanModule) external;
    function setCaliberBeacon(address _caliberBeacon) external;
    function setBridgeAdapterBeacon(uint16 bridgeId, address _bridgeAdapter) external;
    function setBridgeConfig(uint16 bridgeId, address _bridgeConfig) external;
}
```

## IHubCoreRegistry

Hub chain registry extending ICoreRegistry.

```solidity
interface IHubCoreRegistry is ICoreRegistry {
    event ChainRegistryChanged(address indexed oldChainRegistry, address indexed newChainRegistry);
    event MachineBeaconChanged(address indexed oldMachineBeacon, address indexed newMachineBeacon);
    event PreDepositVaultBeaconChanged(address indexed oldPreDepositVaultBeacon, address indexed newPreDepositVaultBeacon);

    function chainRegistry() external view returns (address);
    function machineBeacon() external view returns (address);
    function preDepositVaultBeacon() external view returns (address);

    function setChainRegistry(address _chainRegistry) external;
    function setMachineBeacon(address _machineBeacon) external;
    function setPreDepositVaultBeacon(address _preDepositVaultBeacon) external;
}
```

## ISpokeCoreRegistry

Spoke chain registry extending ICoreRegistry.

```solidity
interface ISpokeCoreRegistry is ICoreRegistry {
    event CaliberMailboxBeaconChanged(address indexed oldCaliberMailboxBeacon, address indexed newCaliberMailboxBeacon);

    function caliberMailboxBeacon() external view returns (address);
    function setCaliberMailboxBeacon(address _caliberMailboxBeacon) external;
}
```

## IChainRegistry

Maps EVM chain IDs to Wormhole chain IDs.

```solidity
interface IChainRegistry {
    event ChainIdsRegistered(uint256 indexed evmChainId, uint16 indexed whChainId);

    function isEvmChainIdRegistered(uint256 _evmChainId) external view returns (bool);
    function isWhChainIdRegistered(uint16 _whChainId) external view returns (bool);
    function evmToWhChainId(uint256 _evmChainId) external view returns (uint16);
    function whToEvmChainId(uint16 _whChainId) external view returns (uint256);
    function setChainIds(uint256 _evmChainId, uint16 _whChainId) external;
}
```

## ITokenRegistry

Maps token addresses across chains.

```solidity
interface ITokenRegistry {
    event TokenRegistered(address indexed localToken, uint256 indexed evmChainId, address indexed foreignToken);

    function getForeignToken(address _localToken, uint256 _foreignEvmChainId) external view returns (address);
    function getLocalToken(address _foreignToken, uint256 _foreignEvmChainId) external view returns (address);
    function setToken(address _localToken, uint256 _foreignEvmChainId, address _foreignToken) external;
}
```

## IOracleRegistry

Chainlink price feed aggregator for token pricing.

```solidity
interface IOracleRegistry {
    event FeedRouteRegistered(address indexed token, address indexed feed1, address indexed feed2);
    event FeedStaleThresholdChanged(address indexed feed, uint256 oldThreshold, uint256 newThreshold);

    struct FeedRoute {
        address feed1;
        address feed2;
    }

    function getFeedStaleThreshold(address feed) external view returns (uint256);
    function isFeedRouteRegistered(address token) external view returns (bool);
    function getFeedRoute(address token) external view returns (address, address);
    function getPrice(address baseToken, address quoteToken) external view returns (uint256);
    function setFeedRoute(
        address token,
        address feed1,
        uint256 stalenessThreshold1,
        address feed2,
        uint256 stalenessThreshold2
    ) external;
    function setFeedStaleThreshold(address feed, uint256 threshold) external;
}
```

## Usage Examples

### Get token price
```solidity
uint256 price = IOracleRegistry(oracleRegistry).getPrice(WETH, USDC);
```

### Map token to foreign chain
```solidity
address foreignToken = ITokenRegistry(tokenRegistry).getForeignToken(localToken, arbitrumChainId);
```

### Convert chain IDs
```solidity
uint16 whChainId = IChainRegistry(chainRegistry).evmToWhChainId(1); // Ethereum mainnet
```
