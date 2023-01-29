|   | Issue List |
|---|---|
| [L-01] | ```MAXPROTOCOLREWARD()``` & ```PROTOCOLFEE()``` CAN ROUND DOWN TO 0 |
| [L-02] | LACK OF ```MSG.SENDER``` CHECKS |
| [L-03] | MISSING ZERO ADDRESS CHECKS |
| [L-04] | USE 2 STEP OWNABLE |
| [NC-01] | LACK OF EMIT EVENTS |
| [NC-02] | OUTDATED COMPILER |
| [NC-03] | SPELLING ERROR |
| [NC-04] | OPEN TODO |

##
# LOW 
---
### [L-01] MAXPROTOCOLREWARD() & PROTOCOLFEE() CAN ROUND DOWN TO 0

**Context:**
[Erc20Quest.sol:52](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L52)
[Erc20Quest.sol:96](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L52)
```javascript
    function maxTotalRewards() public view returns (uint256) {
        return totalParticipants * rewardAmountInWeiOrTokenId;
    }

    function maxProtocolReward() public view returns (uint256) {
        return (maxTotalRewards() * questFee) / 10_000;
    }
```
```javascript
    function receiptRedeemers() public view returns (uint256) {
        return questFactoryContract.getNumberMinted(questId);
    }

    function protocolFee() public view returns (uint256) {
        return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
    }
```

**Description:**
If the totalParticipants, rewardAmountInWeiOrToken & questFee calculation comes out to below 10,000 all instances where the maxProtocolReward() is used will have faulty logic. The same problem is in protocolFee().

**Recommendation:**
Add checks to ensure the the contract will only be constructed if reasonable minimum values for the amounts are given.

##
---
### [L-02] LACK OF MSG.SENDER CHECKS

**Context:**
[RabbitHoleReceipt.sol:109]()
```javascript
    function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
```

**Description:**
Your contracts lack checks almost everywhere, but we'll look at this specific example. Here we have a function that is explicitly defining itself to be dealing with ```msg.sender``` logic by the verbiage ```msgSenderBalance``` yet anyone can insert a claiming address and get the ```balanceOf```. The [Quest.sol:99](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99) may have the msg.sender, but that doesn't make the function safe due to its public declaration. This code block eventually leads to a ```_safeTransfer``` in [Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L66).

**Recommendation:**
```javascript
       + require(msg.sender == claimingAddress_);
       uint msgSenderBalance = balanceOf(claimingAddress_);
```

##
---
### [L-03] MISSING ZERO ADDRESS CHECKS

**Context:**
All contracts and functions fail to add this sanity check besides [QuestFactory.sol:166](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L166)

**Description:**
Adding a zero address sanity check to your methods prevents potentially costly mistakes. Adding the check is an easy and effective way to reduce human error.

**Recommendation:**
```javascript
    require([ADDR_ARGUMENT] != address(0));
```

##
---
### [L-04] USE 2 STEP OWNABLE

**Context:**
[Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L4)

**Description:**
Using a 1 step ownable is not recommended and can cause serious issues if a mistake is made. It's standard practice to make use of a multi step ownable setup instead.

**Recommendation:**
[Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)

##
---
# QA 
---
### [NC-01] LACK OF EMIT EVENTS

**Context:**
[Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57)

**Description:**
Incredibly important functions such as start(), pause() and unPause() lack emit events. When state changes these won't emit information that users and dApps desire.
##
```javascript
    function pause() public onlyOwner onlyStarted {
        isPaused = true;
        emit Pause(isPaused);
    }
```

##
---
### [NC-02] OUTDATED COMPILER
Solidity version 0.8.15 is missing features and bug features listed [here](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

##
---
### [NC-03] SPELLING ERROR
[QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176) remave -> remove

##
---
### [NC-04] OPEN TODO
[IQuest:4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4)

Please ensure that TODOs don't end up being deployed.