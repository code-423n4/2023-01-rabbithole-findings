# 1. USE `SAFETRANSFEROWNERSHIP()` INSTEAD OF `TRANSFEROWNERSHIP()` FUNCTION

## Context

- [QuestFactory.sol#L100](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L100)
- [QuestFactory.sol#L131](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L131)

## Description:

`transferOwnership` function is used to change Ownership from `Owned.sol`.

## Use a 2 structure `transferOwnership` which is safer.

`safeTransferOwnership`, use it is more secure due to the 2-stage ownership transfer.

## Recommendation:

- Use `Ownable2Step.sol`
[Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)



# 2. LOSS OF PRECISION DUE TO ROUNDING

Add scalars so roundings are negligible

## Context

- [Erc20Quest.sol#L53](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L53)
- [RabbitHoleReceipt.sol#L184](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184)

```Solidity
function maxProtocolReward() public view returns (uint256) {
        return (maxTotalRewards() * questFee) / 10_000;
    }

```


# 3. FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

## Context

- [QuestFactory.sol#L10-L11](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L10-L11)
- [RabbitHoleReceipt.sol#L4-L13](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L4-L13)

## Description

Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces to allow us to apply this rule better.

## Recommendation:

`import {contract1 , contract2} from "filename.sol";`

```Solidity
import {Erc20Quest} from './Erc20Quest.sol';
import {IQuestFactory} from './interfaces/IQuestFactory.sol';
import {Erc1155Quest} from './Erc1155Quest.sol';
import {RabbitHoleReceipt} from './RabbitHoleReceipt.sol';
import {OwnableUpgradeable} from '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
```


# 4. USE `BYTES.CONCAT()` INSTEAD OF `ABI.ENCODEPACKED()`

## Context

- [QuestFactory.sol#L72](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72)
- [QuestFactory.sol#L105](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105)
- [QuestFactory.sol#L222](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L222)

```Solidity
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
```

## Recommendation

Rather than using `abi.encodePacked` for appending bytes, since version `0.8.4`, `bytes.concat()` is enabled.
Since version `0.8.4` for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked()`.


# 5. INSUFFICIENT COVERAGE

## Description:
Testing all functions is best practice in terms of security criteria.

This function test coverage is not found in test files.

[RabbitHoleReceipt.sol#L178-L194](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L178-L194)

```Solidity
 function royaltyInfo(uint256 tokenId_, uint256 salePrice_)
        external
        view
        override
        returns (address receiver, uint256 royaltyAmount)
    {
        require(_exists(tokenId_), 'Nonexistent token');

        uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
        return (royaltyRecipient, royaltyPayment);
    }

function supportsInterface(bytes4 interfaceId_)
        public
        view
        virtual
        override(ERC721Upgradeable, ERC721EnumerableUpgradeable, IERC165Upgradeable)
        returns (bool)
    {
        return interfaceId_ == type(IERC2981Upgradeable).interfaceId || super.supportsInterface(interfaceId_);
    }
```

Due to its capacity, test coverage is expected to be 100%.