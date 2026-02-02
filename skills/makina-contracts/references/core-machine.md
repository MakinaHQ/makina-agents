# Core: Machine & Caliber

The main vault and strategy execution contracts.

## IMachine

The primary vault contract managing deposits, redemptions, and cross-chain coordination.

```solidity
interface IMachine is IMachineEndpoint {
    // Events
    event Deposit(address indexed sender, address indexed receiver, uint256 assets, uint256 shares, bytes32 indexed referralKey);
    event Redeem(address indexed owner, address indexed receiver, uint256 assets, uint256 shares);
    event TotalAumUpdated(uint256 totalAum);
    event FeesMinted(uint256 shares);
    event TransferToCaliber(uint256 indexed chainId, address indexed token, uint256 amount);
    event DepositorChanged(address indexed oldDepositor, address indexed newDepositor);
    event RedeemerChanged(address indexed oldRedeemer, address indexed newRedeemer);
    event FeeManagerChanged(address indexed oldFeeManager, address indexed newFeeManager);
    event ShareLimitChanged(uint256 indexed oldShareLimit, uint256 indexed newShareLimit);
    event CaliberStaleThresholdChanged(uint256 indexed oldThreshold, uint256 indexed newThreshold);
    event MaxFixedFeeAccrualRateChanged(uint256 indexed oldMaxAccrualRate, uint256 indexed newMaxAccrualRate);
    event MaxPerfFeeAccrualRateChanged(uint256 indexed oldMaxAccrualRate, uint256 indexed newMaxAccrualRate);
    event FeeMintCooldownChanged(uint256 indexed oldFeeMintCooldown, uint256 indexed newFeeMintCooldown);
    event MaxSharePriceChangeRateChanged(uint256 indexed oldMaxChangeRate, uint256 indexed newMaxChangeRate);
    event SpokeBridgeAdapterSet(uint256 indexed chainId, uint256 indexed bridgeId, address indexed adapter);
    event SpokeCaliberMailboxSet(uint256 indexed chainId, address indexed caliberMailbox);

    struct MachineInitParams {
        address initialDepositor;
        address initialRedeemer;
        address initialFeeManager;
        uint256 initialCaliberStaleThreshold;
        uint256 initialMaxFixedFeeAccrualRate;
        uint256 initialMaxPerfFeeAccrualRate;
        uint256 initialFeeMintCooldown;
        uint256 initialShareLimit;
        uint256 initialMaxSharePriceChangeRate;
    }

    // View functions
    function wormhole() external view returns (address);
    function depositor() external view returns (address);
    function redeemer() external view returns (address);
    function shareToken() external view returns (address);
    function accountingToken() external view returns (address);
    function hubCaliber() external view returns (address);
    function feeManager() external view returns (address);
    function caliberStaleThreshold() external view returns (uint256);
    function maxFixedFeeAccrualRate() external view returns (uint256);
    function maxPerfFeeAccrualRate() external view returns (uint256);
    function feeMintCooldown() external view returns (uint256);
    function shareLimit() external view returns (uint256);
    function maxSharePriceChangeRate() external view returns (uint256);
    function maxMint() external view returns (uint256);
    function maxWithdraw() external view returns (uint256);
    function lastTotalAum() external view returns (uint256);
    function lastGlobalAccountingTime() external view returns (uint256);
    function isIdleToken(address token) external view returns (bool);
    function getIdleTokensLength() external view returns (uint256);
    function getIdleToken(uint256 idx) external view returns (address);
    function getSpokeCalibersLength() external view returns (uint256);
    function getSpokeChainId(uint256 idx) external view returns (uint256);
    function getSpokeCaliberDetailedAum(uint256 chainId) external view returns (uint256 aum, bytes[] memory positions, bytes[] memory baseTokens, uint256 timestamp);
    function getSpokeCaliberMailbox(uint256 chainId) external view returns (address);
    function getSpokeBridgeAdapter(uint256 chainId, uint16 bridgeId) external view returns (address);
    function convertToShares(uint256 assets) external view returns (uint256);
    function convertToAssets(uint256 shares) external view returns (uint256);

    // State-changing functions
    function deposit(uint256 assets, address receiver, uint256 minShares, bytes32 referralKey) external returns (uint256);
    function redeem(uint256 shares, address receiver, uint256 minAssets) external returns (uint256);
    function updateTotalAum() external returns (uint256);
    function transferToHubCaliber(address token, uint256 amount) external;
    function transferToSpokeCaliber(uint16 bridgeId, uint256 chainId, address token, uint256 amount, uint256 minOutputAmount) external;
    function updateSpokeCaliberAccountingData(bytes memory response, GuardianSignature[] memory signatures) external;
    function setSpokeCaliber(uint256 chainId, address spokeCaliberMailbox, uint16[] calldata bridges, address[] calldata adapters) external;
    function setDepositor(address newDepositor) external;
    function setRedeemer(address newRedeemer) external;
    function setFeeManager(address newFeeManager) external;
    function setShareLimit(uint256 newShareLimit) external;
    function setCaliberStaleThreshold(uint256 newCaliberStaleThreshold) external;
    function setMaxFixedFeeAccrualRate(uint256 newMaxAccrualRate) external;
    function setMaxPerfFeeAccrualRate(uint256 newMaxAccrualRate) external;
    function setFeeMintCooldown(uint256 newFeeMintCooldown) external;
    function setMaxSharePriceChangeRate(uint256 newMaxSharePriceChangeRate) external;
    function setSpokeBridgeAdapter(uint256 chainId, uint16 bridgeId, address adapter) external;
}
```

## IMachineShare

ERC20 token representing shares in the Machine vault.

```solidity
interface IMachineShare is IERC20Metadata {
    function minter() external view returns (address);
    function mint(address to, uint256 amount) external;
    function burn(address from, uint256 amount) external;
}
```

## ICaliber

Strategy execution contract managing positions.

```solidity
interface ICaliber {
    // Events
    event PositionCreated(uint256 indexed id, uint256 value);
    event PositionUpdated(uint256 indexed id, uint256 value);
    event PositionClosed(uint256 indexed id);
    event BaseTokenAdded(address indexed token);
    event BaseTokenRemoved(address indexed token);
    event TransferToHubMachine(address indexed token, uint256 amount);
    event IncomingTransfer(address indexed token, uint256 amount);
    event NewAllowedInstrRootScheduled(bytes32 indexed newMerkleRoot, uint256 indexed effectiveTime);
    event NewAllowedInstrRootCancelled(bytes32 indexed cancelledMerkleRoot);
    event InstrRootGuardianAdded(address indexed newGuardian);
    event InstrRootGuardianRemoved(address indexed guardian);
    event PositionStaleThresholdChanged(uint256 indexed oldThreshold, uint256 indexed newThreshold);
    event TimelockDurationChanged(uint256 indexed oldDuration, uint256 indexed newDuration);
    event MaxPositionIncreaseLossBpsChanged(uint256 indexed oldBps, uint256 indexed newBps);
    event MaxPositionDecreaseLossBpsChanged(uint256 indexed oldBps, uint256 indexed newBps);
    event MaxSwapLossBpsChanged(uint256 indexed oldBps, uint256 indexed newBps);
    event CooldownDurationChanged(uint256 indexed oldDuration, uint256 indexed newDuration);

    enum InstructionType {
        MANAGEMENT,
        ACCOUNTING,
        HARVEST,
        FLASHLOAN_MANAGEMENT
    }

    struct CaliberInitParams {
        uint256 initialPositionStaleThreshold;
        bytes32 initialAllowedInstrRoot;
        uint256 initialTimelockDuration;
        uint256 initialMaxPositionIncreaseLossBps;
        uint256 initialMaxPositionDecreaseLossBps;
        uint256 initialMaxSwapLossBps;
        uint256 initialCooldownDuration;
    }

    struct Instruction {
        uint256 positionId;
        bool isDebt;
        uint256 groupId;
        InstructionType instructionType;
        address[] affectedTokens;
        address[] positionTokens;
        bytes32[] commands;
        bytes[] state;
        uint128 stateBitmap;
        bytes32[] merkleProof;
    }

    struct Position {
        uint256 lastAccountingTime;
        uint256 value;
        bool isDebt;
    }

    // View functions
    function weirollVm() external view returns (address);
    function hubMachineEndpoint() external view returns (address);
    function accountingToken() external view returns (address);
    function positionStaleThreshold() external view returns (uint256);
    function allowedInstrRoot() external view returns (bytes32);
    function timelockDuration() external view returns (uint256);
    function pendingAllowedInstrRoot() external view returns (bytes32);
    function pendingTimelockExpiry() external view returns (uint256);
    function maxPositionIncreaseLossBps() external view returns (uint256);
    function maxPositionDecreaseLossBps() external view returns (uint256);
    function maxSwapLossBps() external view returns (uint256);
    function cooldownDuration() external view returns (uint256);
    function getPositionsLength() external view returns (uint256);
    function getPositionId(uint256 idx) external view returns (uint256);
    function getPosition(uint256 id) external view returns (Position memory);
    function isBaseToken(address token) external view returns (bool);
    function getBaseTokensLength() external view returns (uint256);
    function getBaseToken(uint256 idx) external view returns (address);
    function isInstrRootGuardian(address user) external view returns (bool);
    function isAccountingFresh() external view returns (bool);
    function getDetailedAum() external view returns (uint256 netAum, bytes[] memory positions, bytes[] memory baseTokens);

    // State-changing functions
    function addBaseToken(address token) external;
    function removeBaseToken(address token) external;
    function accountForPosition(Instruction calldata instruction) external returns (uint256 value, int256 change);
    function accountForPositionBatch(Instruction[] calldata instructions, uint256[] calldata groupIds) external returns (uint256[] memory values, int256[] memory changes);
    function managePosition(Instruction calldata mgmtInstruction, Instruction calldata acctInstruction) external returns (uint256 value, int256 change);
    function managePositionBatch(Instruction[] calldata mgmtInstructions, Instruction[] calldata acctInstructions) external returns (uint256[] memory values, int256[] memory changes);
    function manageFlashLoan(Instruction calldata instruction, address token, uint256 amount) external;
    function harvest(Instruction calldata instruction, ISwapModule.SwapOrder[] calldata swapOrders) external;
    function swap(ISwapModule.SwapOrder calldata order) external;
    function transferToHubMachine(address token, uint256 amount, bytes calldata data) external;
    function notifyIncomingTransfer(address token, uint256 amount) external;
    function scheduleAllowedInstrRootUpdate(bytes32 newMerkleRoot) external;
    function cancelAllowedInstrRootUpdate() external;
    function setPositionStaleThreshold(uint256 newPositionStaleThreshold) external;
    function setTimelockDuration(uint256 newTimelockDuration) external;
    function setMaxPositionIncreaseLossBps(uint256 newMaxPositionIncreaseLossBps) external;
    function setMaxPositionDecreaseLossBps(uint256 newMaxPositionDecreaseLossBps) external;
    function setMaxSwapLossBps(uint256 newMaxSwapLossBps) external;
    function setCooldownDuration(uint256 newCooldownDuration) external;
    function addInstrRootGuardian(address newGuardian) external;
    function removeInstrRootGuardian(address guardian) external;
}
```

## ICaliberMailbox

Spoke chain mailbox coordinating with hub Machine.

```solidity
interface ICaliberMailbox is IMachineEndpoint {
    event CaliberSet(address indexed caliber);
    event HubBridgeAdapterSet(uint256 indexed bridgeId, address indexed adapter);
    event CooldownDurationChanged(uint256 oldDuration, uint256 newDuration);

    struct SpokeCaliberAccountingData {
        uint256 netAum;
        bytes[] positions;
        bytes[] baseTokens;
        bytes[] bridgesIn;
        bytes[] bridgesOut;
    }

    function caliber() external view returns (address);
    function cooldownDuration() external view returns (uint256);
    function getHubBridgeAdapter(uint16 bridgeId) external view returns (address);
    function hubChainId() external view returns (uint256);
    function getSpokeCaliberAccountingData() external view returns (SpokeCaliberAccountingData memory);
    function setCaliber(address caliber) external;
    function setCooldownDuration(uint256 newCooldownDuration) external;
    function setHubBridgeAdapter(uint16 bridgeId, address adapter) external;
}
```

## IMachineEndpoint

Base interface for Machine and CaliberMailbox.

```solidity
interface IMachineEndpoint is IBridgeController, IMakinaGovernable {
    function manageTransfer(address token, uint256 amount, bytes calldata data) external;
}
```

## IPreDepositVault

Vault for collecting deposits before Machine launch.

```solidity
interface IPreDepositVault {
    event Deposit(address indexed sender, address indexed receiver, uint256 assets, uint256 shares, bytes32 indexed referralKey);
    event Redeem(address indexed owner, address indexed receiver, uint256 assets, uint256 shares);
    event MigrateToMachine(address indexed machine);

    struct PreDepositVaultInitParams {
        uint256 initialShareLimit;
        bool initialWhitelistMode;
        address initialRiskManager;
        address initialAuthority;
    }

    function migrated() external view returns (bool);
    function machine() external view returns (address);
    function riskManager() external view returns (address);
    function whitelistMode() external view returns (bool);
    function isWhitelistedUser(address user) external view returns (bool);
    function depositToken() external view returns (address);
    function accountingToken() external view returns (address);
    function shareToken() external view returns (address);
    function shareLimit() external view returns (uint256);
    function maxDeposit() external view returns (uint256);
    function totalAssets() external view returns (uint256);
    function previewDeposit(uint256 assets) external view returns (uint256);
    function previewRedeem(uint256 assets) external view returns (uint256);
    function deposit(uint256 assets, address receiver, uint256 minShares, bytes32 referralKey) external returns (uint256);
    function redeem(uint256 shares, address receiver, uint256 minAssets) external returns (uint256);
    function migrateToMachine() external;
    function setPendingMachine(address machine) external;
    function setShareLimit(uint256 newShareLimit) external;
    function setWhitelistedUsers(address[] calldata users, bool whitelisted) external;
    function setWhitelistMode(bool enabled) external;
}
```

## IMakinaGovernable

Governance role management with accounting agent permissions.

```solidity
interface IMakinaGovernable {
    event MechanicChanged(address indexed oldMechanic, address indexed newMechanic);
    event SecurityCouncilChanged(address indexed oldSecurityCouncil, address indexed newSecurityCouncil);
    event RiskManagerChanged(address indexed oldRiskManager, address indexed newRiskManager);
    event RiskManagerTimelockChanged(address indexed oldRiskManagerTimelock, address indexed newRiskManagerTimelock);
    event RecoveryModeChanged(bool recoveryMode);
    event RestrictedAccountingModeChanged(bool restrictedAccountingMode);
    event AccountingAgentAdded(address indexed newAgent);
    event AccountingAgentRemoved(address indexed agent);

    struct MakinaGovernableInitParams {
        address initialMechanic;
        address initialSecurityCouncil;
        address initialRiskManager;
        address initialRiskManagerTimelock;
        address initialAuthority;
        bool initialRestrictedAccountingMode;
    }

    // View functions
    function mechanic() external view returns (address);
    function securityCouncil() external view returns (address);
    function riskManager() external view returns (address);
    function riskManagerTimelock() external view returns (address);
    function recoveryMode() external view returns (bool);
    function restrictedAccountingMode() external view returns (bool);
    function isAccountingAgent(address agent) external view returns (bool);
    function isOperator(address user) external view returns (bool);
    function isAccountingAuthorized(address user) external view returns (bool);

    // State-changing functions
    function setMechanic(address newMechanic) external;
    function setSecurityCouncil(address newSecurityCouncil) external;
    function setRiskManager(address newRiskManager) external;
    function setRiskManagerTimelock(address newRiskManagerTimelock) external;
    function setRecoveryMode(bool enabled) external;
    function setRestrictedAccountingMode(bool enabled) external;
    function addAccountingAgent(address newAgent) external;
    function removeAccountingAgent(address agent) external;
}
```

### Accounting Authorization

The `isAccountingAuthorized` function determines if an address can perform accounting operations:
- If `restrictedAccountingMode` is false: returns true for mechanic or riskManager
- If `restrictedAccountingMode` is true: returns true only for designated accounting agents

```solidity
// Check who can do accounting
bool canAccount = IMakinaGovernable(machine).isAccountingAuthorized(caller);

// Enable restricted mode (security council only)
IMakinaGovernable(machine).setRestrictedAccountingMode(true);

// Add authorized accounting agent
IMakinaGovernable(machine).addAccountingAgent(botAddress);
```
