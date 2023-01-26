### Low Risk Items
| Number |Issues Details|
|:--:|:-------|
|[L-01]| _disableInitializers not used in QuestFactory constructor |
|[L-02]| No zero address check for `withdrawRemainingTokens` |

### Non-Critical Items
| Number |Issues Details|
|:--:|:-------|
|[NC-01]| TODO left in code |
|[NC-02]| Comparison to boolean constants |


## L-01 _disableInitializers not used in QuestFactory constructor

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

The constructor in QuestFactory uses the `initializer` modifier to block initialization in the logic contract. This is no longer the recommended method for doing so. Current guidance from OpenZeppelin is to use `_disableInitialzer` in the constructor's body to ensure that the contract's initialization version is set to the maximum value. 

    /**
     * @dev Locks the contract, preventing any future reinitialization. This cannot be part of an initializer call.
     * Calling this in the constructor of a contract will prevent that contract from being initialized or reinitialized
     * to any version. It is recommended to use this to lock implementation contracts that are designed to be called
     * through proxies.
     *
     * Emits an {Initialized} event the first time it is successfully executed.
     */
    function _disableInitializers() internal virtual {
        require(!_initializing, "Initializable: contract is initializing");
        if (_initialized < type(uint8).max) {
            _initialized = type(uint8).max;
            emit Initialized(type(uint8).max);
        }
    }

The constructors for RabbitHoleReceipt and RabbitHoleTickets both have the correct usage:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L39-L41

    constructor() { 
        _disableInitializers();
    }

It's recommended to use the `_disableInitializers` function over the `initializer` modifier for the implementation contract's constructor.

## L-02 No zero address check for `withdrawRemainingTokens`

This is an important function to check for zero address input since you don't want to accidentally burn all the remaining tokens when attempting to withdraw.

Erc1155Quest:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54-L63

Erc20Quest:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87

Quest:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150

Recommend adding a zero address check to the instance in Quest.sol since it's called in both overriding functions in their respective contracts.

Possible implementation:

    function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {
        if(to_ == address(0)) revert ZeroAddress();
    }

## NC-01 TODO left in code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4

Don't forget to remove this before deployment

## NC-02 Comparison to boolean constants

Convention is to avoid comparisons to boolean constants

Instances:

1. https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136

2. https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73

3. https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221

`== true` is superfluous as the expression will evaluate to `true` only if the boolean is already `true`.

    return claimedList[tokenId_] == true;

is the same as 

    return claimedList[tokenId_];

Likewise, `== false` can be replaced by the negation operator `!`

    if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

is the same as 

    if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();


