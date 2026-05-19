# Flash Loans

`FlashLoanModule` is the gateway between MakinaLite modules and external flash loan providers. Today only [Morpho](https://morpho.org/) is supported; other enum entries in `FlashLoanProvider` are preserved for stable indexing.

## IFlashLoanModule

```solidity
interface IFlashLoanModule is IMorphoFlashLoanCallback {
    enum FlashLoanProvider {
        DEPRECATED_0,
        DEPRECATED_1,
        DEPRECATED_2,
        MORPHO,
        DEPRECATED_3
    }

    struct FlashLoanRequest {
        address taker;                               // The MakinaLiteModule that will receive the loan
        FlashLoanProvider provider;                  // Currently MORPHO
        IWeirollComponent.Instruction instruction;   // FLASHLOAN_MANAGEMENT instruction to execute
        address token;
        uint256 amount;
    }

    function requestFlashLoan(FlashLoanRequest calldata request) external;
}

interface IMorphoFlashLoanCallback {
    function onMorphoFlashLoan(uint256 assets, bytes calldata data) external;
}
```

## Authorization

`requestFlashLoan` requires:
- `taker` is a module deployed by `ModuleFactory` (`isMakinaLiteModule(taker) == true`).
- `msg.sender == IMakinaLiteModule(taker).safe()` — i.e. the Safe that owns the module is the caller.

In other words, only the Safe of a factory-deployed module can request a flash loan on behalf of that module.

## Flow

```
Safe (msg.sender)
  └── FlashLoanModule.requestFlashLoan(request)
        ├── validates taker is a factory-deployed module, caller is its Safe
        ├── stores keccak256(data) in transient storage (single-use)
        └── Morpho.flashLoan(token, amount, data)
              └── Morpho ──► FlashLoanModule.onMorphoFlashLoan(assets, data)
                    ├── reverts unless msg.sender == morpho
                    ├── consumes (validates + clears) transient data hash
                    ├── approves taker module for `assets`
                    ├── taker.manageFlashLoan(instruction, token, amount)
                    │     └── module pulls funds from FlashLoanModule, executes
                    │        the FLASHLOAN_MANAGEMENT instruction via Safe,
                    │        and instructs the Safe to repay
                    └── increases Morpho's allowance for `assets` to enable repayment
```

The transient `EXPECTED_DATA_HASH_SLOT` (ERC-7201 namespaced) acts as a reentrancy guard: the request encodes `(token, taker, instruction)` and only the matching callback can consume it.

## Module-side: manageFlashLoan

```solidity
// IWeirollComponent
function manageFlashLoan(Instruction calldata instruction, address token, uint256 amount) external;
```

Constraints:
- Called by `FlashLoanModule` during the Morpho callback only.
- `instruction.instructionType` must be `FLASHLOAN_MANAGEMENT`.
- Only callable inside an outer `managePosition` so that the parent `MANAGEMENT` instruction's `affectedTokens` defines the allowed balance-change set.
- Must not leave persistent ERC20 approvals from the Safe.

## Deployment

`FlashLoanModule` is constructed with `(moduleFactory, morpho)` and is non-upgradeable. Its address is registered in `MakinaLiteRegistry.flashLoanModule()`.

## Sources

- `src/flash-loans/FlashLoanModule.sol`
- `src/interfaces/IFlashLoanModule.sol`
- `src/interfaces/IMorpho.sol`
- `src/interfaces/IMorphoFlashLoanCallback.sol`
