# 1. Do not check `address(0)`
## Impact
I checked functions in the Prime.sol file and found that these functions do not check address(0) of input. Including  `(address[] memory users)` parameter.

If you don't check for user and user is an invalid address (0), you may cause a logic error in the function, because there may be no valid data to calculate or query.

No need to check `address(0)` for internal functions. Checking should inherently be checked in public/external functions.

If the function is called from a contract or a user interface, checking the user can help avoid unnecessary errors when calling the function. If an invalid `address(0)` is passed in, the function may return an error or not make any changes.

## Proof Of Concept
https://github.com/code-423n4/2023-09-venus/blame/9c1016326a0e97376749860266c4541a313a86c2/contracts/Tokens/Prime/Prime.sol#L174
```
    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        ...
    }
```

https://github.com/code-423n4/2023-09-venus/blame/9c1016326a0e97376749860266c4541a313a86c2/contracts/Tokens/Prime/Prime.sol#L200
```
    function updateScores(address[] memory users) external {
        ...
    }
```

https://github.com/code-423n4/2023-09-venus/blame/9c1016326a0e97376749860266c4541a313a86c2/contracts/Tokens/Prime/Prime.sol#L331-L331C76
```
    function issue(bool isIrrevocable, address[] calldata users) external {
        ...
    }
```

https://github.com/code-423n4/2023-09-venus/blame/9c1016326a0e97376749860266c4541a313a86c2/contracts/Tokens/Prime/Prime.sol#L365
```
    function xvsUpdated(address user) external {
        ...
    }
```

https://github.com/code-423n4/2023-09-venus/blame/9c1016326a0e97376749860266c4541a313a86c2/contracts/Tokens/Prime/Prime.sol#L452
```
    function updateAssetsState(address _comptroller, address asset) external {
        ...
    }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L554
```
    function accrueInterest(address vToken) public {}
```

## Recommendation
Use a modifier responsible for checking for `address(0)`. 

# 2. Function state mutability can be restricted to pure

Some internal functions can become pure functions.

`view` function promise not to modify the state.

`pure` function promise not to modify or read from the state.

## Proof Of Concept

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L809

```
   function _checkAlphaArguments(uint128 _alphaNumerator, uint128 _alphaDenominator) internal {
        ...
    }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L854

```
    function _xvsBalanceForScore(uint256 xvs) internal view returns (uint256) {
        ...
    }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L904C1-L910C6

```
    function isEligible(uint256 amount) internal view returns (bool) {
        ...
    }
```

## Recommendation
Add or edit the function to `pure` functions.

# 3. Use a newer version of solidity

When you upgrade the Solidity version of an interface to a newer version, you can still use that interface in contracts with the older Solidity version.

The Solidity source code of an interface specifies how other contracts can interact with it, and the Solidity version of the contract using the interface need not match the Solidity version of the interface. This means that contracts with older Solidity versions can still call functions from interfaces that have been upgraded to newer Solidity versions without any changes to the old contract.

## Proof Of Concept

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/IPrime.sol#L2C1-L2C25

```
pragma solidity ^0.5.16;
```

## Recommendation
Can be edited to version `^0.8.0` or `0.8.13`

# 4. Declaration shadows an existing declaration

You have declared the variable `pendingInterests` in the function `getPendingInterests`, but this variable name has been used above in the function's return type declaration.

## Proof Of Concept
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L174
```
    function getPendingInterests(address user) external returns (PendingInterest[] memory) {
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
                market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }

        return pendingInterests;
    }
```

## Recommendation
Here, the return variable name can be deleted, to avoid conflicts with the local variable pendingInterests.

# 5. Advice on folder structure

I don't know why the interface file is outside the interfaces folder. Maybe your purpose is to easily find this interface. However I still want to refactor it.

## Recommendation
```
    ...
    └── contracts/
        ├── interfaces/
              ├──IPrime.sol
        └── libs/
            ...
        ...    
```

# 6. Advice on contracts file structure

I don't know much about your sorting method. I see that your sorting logic does not divide functions according to their visibility, nor according to the general function of the functions. So below is how I structured the function according to its visibility.

The way I see your project is that the two files `Prime.sol` and the file `PrimeLiquidityProvider.sol` do not have an inconsistent arrangement of the file structure.

## Proof Of Concept
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

## Recommendation
Note: I have deleted all comments and unnecessary things just to keep this report from being too long.

For structuring contracts according to functions visibility.
```
// SPDX-License-Identifier: BSD-3-Clause
pragma solidity 0.8.13;

import {SafeERC20Upgradeable, IERC20Upgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
import {AccessControlledV8} from "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
import {ResilientOracleInterface} from "@venusprotocol/oracle/contracts/interfaces/OracleInterface.sol";
import {PausableUpgradeable} from "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
import {MaxLoopsLimitHelper} from "@venusprotocol/isolated-pools/contracts/MaxLoopsLimitHelper.sol";

import {PrimeStorageV1} from "./PrimeStorage.sol";
import {Scores} from "./libs/Scores.sol";

import {IPrimeLiquidityProvider} from "./Interfaces/IPrimeLiquidityProvider.sol";
import {IXVSVault} from "./Interfaces/IXVSVault.sol";
import {IVToken} from "./Interfaces/IVToken.sol";
import {IProtocolShareReserve} from "./Interfaces/IProtocolShareReserve.sol";
import {IIncomeDestination} from "./Interfaces/IIncomeDestination.sol";
import {InterfaceComptroller} from "./Interfaces/InterfaceComptroller.sol";

contract Prime is
    IIncomeDestination,
    AccessControlledV8,
    PausableUpgradeable,
    MaxLoopsLimitHelper,
    PrimeStorageV1
{
    //////////////////////////////////////////////////
    //////////////// Errors /////////////////////////
    ////////////////////////////////////////////////
    error MarketNotSupported();
    error InvalidLimit();
    error IneligibleToClaim();
    error WaitMoreTime();
    error UserHasNoPrimeToken();
    error InvalidCaller();
    error InvalidComptroller();
    error NoScoreUpdatesRequired();
    error MarketAlreadyExists();
    error InvalidAddress();
    error InvalidBlocksPerYear();
    error InvalidAlphaArguments();
    error InvalidVToken();

    //////////////////////////////////////////////////
    //////////////// Types //////////////////////////
    ////////////////////////////////////////////////
    using SafeERC20Upgradeable for IERC20Upgradeable;

    //////////////////////////////////////////////////
    //////////////// State Variables ////////////////
    ////////////////////////////////////////////////
    uint256 public immutable BLOCKS_PER_YEAR;
    address public immutable WBNB;
    address public immutable VBNB;

    //////////////////////////////////////////////////
    //////////////// Events /////////////////////////
    ////////////////////////////////////////////////
    event Mint(address indexed user, bool isIrrevocable);

    event Burn(address indexed user);

    event UpdatedAssetsState(
        address indexed comptroller,
        address indexed asset
    );

    event MarketAdded(
        address indexed market,
        uint256 indexed supplyMultiplier,
        uint256 indexed borrowMultiplier
    );

    event MintLimitsUpdated(
        uint256 indexed oldIrrevocableLimit,
        uint256 indexed oldRevocableLimit,
        uint256 indexed newIrrevocableLimit,
        uint256 newRevocableLimit
    );

    event UserScoreUpdated(address indexed user);

    event AlphaUpdated(
        uint128 indexed oldNumerator,
        uint128 indexed oldDenominator,
        uint128 indexed newNumerator,
        uint128 newDenominator
    );

    event MultiplierUpdated(
        address indexed market,
        uint256 indexed oldSupplyMultiplier,
        uint256 indexed oldBorrowMultiplier,
        uint256 newSupplyMultiplier,
        uint256 newBorrowMultiplier
    );

    event InterestClaimed(
        address indexed user,
        address indexed market,
        uint256 amount
    );

    event TokenUpgraded(address indexed user);

    //////////////////////////////////////////////////
    //////////////// Constuctor /////////////////////
    ////////////////////////////////////////////////
    constructor(address _wbnb, address _vbnb, uint256 _blocksPerYear) {}

    //////////////////////////////////////////////////
    //////////////// External Funtions //////////////
    ////////////////////////////////////////////////
    function initialize() external virtual initializer {}

    function updateMultipliers() external {}

    function addMarket() external {}

    function setLimit() external {}

    function issue(bool isIrrevocable, address[] calldata users) external {}

    function xvsUpdated(address user) external {}

    function accrueInterestAndUpdateScore() external {}

    function claim() external {}

    function burn(address user) external {}

    function togglePause() external {}

    function claimInterest() external whenNotPaused returns (uint256) {}

    function updateAssetsState(address _comptroller, address asset) external {}

    //////////////////////////////////////////////////
    ////////////// Public Functions//////////////////
    ////////////////////////////////////////////////
    function accrueInterest(address vToken) public {}

    function getInterestAccrued() public returns (uint256) {}

    //////////////////////////////////////////////////
    ////////////// Internal Functions////////////////
    ////////////////////////////////////////////////
    function _accrueInterestAndUpdateScore(address user) internal {}

    function _initializeMarkets(address account) internal {}

    function _calculateScore() internal returns (uint256) {}

    function _claimInterest() internal returns (uint256) {}

    function _mint(bool isIrrevocable, address user) internal {}

    function _burn(address user) internal {}

    function _upgrade(address user) internal {}

    function _executeBoost(address user, address vToken) internal {}

    function _updateScore(address user, address market) internal {}

    function _startScoreUpdateRound() internal {}

    function _updateRoundAfterTokenBurned(address user) internal {}

    //////////////////////////////////////////////////
    ////// Internal View/Pure Funtions //////////////
    ////////////////////////////////////////////////
    function _checkAlphaArguments() internal pure {}

    function _xvsBalanceOfUser(address user) internal view returns (uint256) {}

    function _xvsBalanceForScore(uint256 xvs) internal pure returns (uint256) {}

    function _capitalForScore() internal view returns () {}

    function _isEligible(uint256 amount) internal pure returns (bool) {}

    function _interestAccrued() internal view returns (uint256) {}

    function _getUnderlying(address vToken) internal view returns (address) {}

    function _incomePerBlock(address vToken) internal view returns (uint256) {}

    function _distributionPercentage() internal view returns (uint256) {}

    function _incomeDistributionYearly() internal view returns () {}

    function _calculateUserAPR() internal view returns () {}

    //////////////////////////////////////////////////
    ////// External View/Pure Funtions //////////////
    ////////////////////////////////////////////////
    function getAllMarkets() external view returns (address[] memory) {}

    function claimTimeRemaining(address user) external view returns (uint256) {}

    function calculateAPR() external view {}

    function estimateAPR() external view {}
}

```
All of the structures I gave you as examples above are suggestions.

I think your project needs to follow a common convention for how to organize the folder structure, as well as a general convention for how to structure the project. So when a new person joins the team, just look at the structure to easily find what they need. As well as serving future audits.

You may want to have an Interface to structure all external functions, errors and events in it more clearly. However, I also understand that you may want other protocols, or other smart contracts, to have the right to call what is in the Prime contract once it is deployed by expressing it in the IPrime.sol file.

One more suggestion on how to organize files:

Layout of Contract:
- Version
- Imports
- Errors
- Interfaces, libraries
- Type declarations
- State variables
- Events
- Modifiers
- Functions

Layout of Funtions:
- Constructor
- Receive funtion (if exists)
- Fallback function (if exists)
- External
- Public
- Internal
- Private
- Internal & Private View/Pure Functions
- External & Public View & Pure Functions

# 7. Edit the name of the internal function.
For internal functions, I understand that your code uses hyphens in front of the function name, but here it is not followed.

## Proof Of Concept
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L904

```
    function isEligible(uint256 amount) internal view returns (bool) {}
```

## Recommendation
```
    function _isEligible(uint256 amount) internal view returns (bool) {}
```