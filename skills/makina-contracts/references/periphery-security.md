# Periphery: Security Module

Security module for protocol insurance through staked shares.

## ISecurityModule

ERC20 token representing staked MachineShares with slashing capability.

```solidity
interface ISecurityModule is IERC20Metadata, IMachinePeriphery {
    // Events
    event Lock(address indexed account, address indexed receiver, uint256 assets, uint256 shares);
    event Cooldown(uint256 indexed cooldownId, address indexed account, address indexed receiver, uint256 shares, uint256 maturity);
    event CooldownCancelled(uint256 indexed cooldownId, address indexed receiver, uint256 shares);
    event Redeem(uint256 indexed cooldownId, address indexed receiver, uint256 assets, uint256 shares);
    event Slash(uint256 amount);
    event SlashingSettled();
    event CooldownDurationChanged(uint256 oldCooldownDuration, uint256 newCooldownDuration);
    event MaxSlashableBpsChanged(uint256 oldMaxSlashableBps, uint256 newMaxSlashableBps);
    event MinBalanceAfterSlashChanged(uint256 oldMinBalanceAfterSlash, uint256 newMinBalanceAfterSlash);

    struct SecurityModuleInitParams {
        address machineShare;
        uint256 initialCooldownDuration;
        uint256 initialMaxSlashableBps;
        uint256 initialMinBalanceAfterSlash;
    }

    struct PendingCooldown {
        uint256 shares;
        uint256 maxAssets;
        uint256 maturity;
    }

    // View functions
    function machineShare() external view returns (address);
    function cooldownReceipt() external view returns (address);
    function cooldownDuration() external view returns (uint256);
    function maxSlashableBps() external view returns (uint256);
    function minBalanceAfterSlash() external view returns (uint256);
    function pendingCooldown(uint256 cooldownId) external view returns (uint256 shares, uint256 currentExpectedAssets, uint256 maturity);
    function slashingMode() external view returns (bool);
    function totalLockedAmount() external view returns (uint256);
    function maxSlashable() external view returns (uint256);
    function convertToShares(uint256 assets) external view returns (uint256 shares);
    function convertToAssets(uint256 shares) external view returns (uint256 assets);
    function previewLock(uint256 assets) external view returns (uint256 shares);

    // User functions
    function lock(uint256 assets, address receiver, uint256 minShares) external returns (uint256 shares);
    function startCooldown(uint256 shares, address receiver) external returns (uint256 cooldownId, uint256 maxAssets, uint256 maturity);
    function cancelCooldown(uint256 cooldownId) external returns (uint256 shares);
    function redeem(uint256 cooldownId, uint256 minAssets) external returns (uint256 assets);

    // Admin functions
    function slash(uint256 amount) external;
    function settleSlashing() external;
    function setCooldownDuration(uint256 cooldownDuration) external;
    function setMaxSlashableBps(uint256 maxSlashableBps) external;
    function setMinBalanceAfterSlash(uint256 minBalanceAfterSlash) external;
}
```

## ISMCooldownReceipt

NFT representing a pending cooldown withdrawal.

```solidity
interface ISMCooldownReceipt is IERC721 {
    function nextTokenId() external view returns (uint256);
    function mint(address to) external returns (uint256 tokenId);
    function burn(uint256 tokenId) external;
}
```

## How Security Module Works

### Locking (Staking)
Users lock MachineShare tokens in the Security Module to earn additional yield and provide protocol insurance.

```solidity
// 1. Approve MachineShares
IERC20(machineShare).approve(securityModule, assets);

// 2. Lock shares and receive SM tokens
uint256 smShares = ISecurityModule(securityModule).lock(
    1000e18,        // MachineShare amount
    msg.sender,     // receiver of SM tokens
    990e18          // min SM tokens to receive
);
```

### Unlocking (Cooldown)
Withdrawals require a cooldown period before assets can be claimed.

```solidity
// 1. Start cooldown - receives NFT receipt
(uint256 cooldownId, uint256 maxAssets, uint256 maturity) =
    ISecurityModule(securityModule).startCooldown(
        smShares,       // SM token amount
        msg.sender      // receiver of cooldown NFT
    );

// 2. Wait for maturity (cooldown duration)

// 3. Redeem after cooldown
uint256 assets = ISecurityModule(securityModule).redeem(
    cooldownId,
    minAssets
);
```

### Cancelling Cooldown
```solidity
// Return SM tokens and burn cooldown NFT
uint256 shares = ISecurityModule(securityModule).cancelCooldown(cooldownId);
```

## Slashing

In case of protocol losses, the Security Module can be slashed:

```solidity
// Only security council can slash
ISecurityModule(securityModule).slash(slashAmount);

// Settle slashing (distributes losses to stakers)
ISecurityModule(securityModule).settleSlashing();
```

### Slashing Parameters
- `maxSlashableBps`: Maximum percentage that can be slashed (e.g., 3000 = 30%)
- `minBalanceAfterSlash`: Minimum balance to maintain after slashing

## Security Module Tokens

The Security Module issues its own ERC20 tokens (SM tokens):
- 1:1 with MachineShares when no slashing has occurred
- Less than 1:1 after a slashing event
- SM tokens can be traded but represent claims on underlying MachineShares

## Benefits of Staking

1. **Higher yield**: SM stakers may receive additional protocol revenue
2. **Protocol governance**: May include voting rights
3. **Priority in losses**: SM absorbs losses before regular shareholders

## Cooldown Receipt NFT

When starting a cooldown:
- User receives an ERC721 NFT
- NFT represents the pending withdrawal
- NFT is transferable (can sell withdrawal position)
- NFT is burned upon redemption
