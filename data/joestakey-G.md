# Gas Report

## Table of Contents

### Gas saving issues
- [Redundant getters](#redundant-getters)
- [`timestampForTokenId` can be removed](#timestampForTokenId-can-be-removed)
- [`setQuestFactory` and `setMinter` can be combined in one function](#setQuestFactory-and-setMinter-can-be-combined-in-one-function)




## Summary

> A few optimizations allow to save significant amounts of gas upon `QuestFactory.createQuest()` and `QuestFactory.mintReceipt()` calls, ie the most important user-facing functions of the protocol.
> The total gas saved on users function calls is `41,852`
> The total gas saved on admin function calls is `21,000`
> The total gas saved on deployment is `81,841`


# Redundant getters


Getters are automatically defined for public storage variables and constants.
The functions `Quest.getRewardAmount()` and `Quest.getRewardToken()` are hence redundant and can be replaced to save gas upon deploying new Quests - ie when calling `QuestFactory.createQuest()`

## Instances

### Quest.sol

```solidity
139:      /// @dev Returns the reward amount
140:      function getRewardAmount() public view returns (uint256) {
141:          return rewardAmountInWeiOrTokenId;
142:      }
143: 
144:      /// @dev Returns the reward token address
145:      function getRewardToken() public view returns (address) {
146:          return rewardToken;
147:      }
```

## Tools Used

Manual Analysis, Hardhat

## Mitigation

```diff
-139:      /// @dev Returns the reward amount
-140:      function getRewardAmount() public view returns (uint256) -{
-141:          return rewardAmountInWeiOrTokenId;
-142:      }
-143: 
-144:      /// @dev Returns the reward token address
-145:      function getRewardToken() public view returns (address) {
-146:          return rewardToken;
-147:      }
```

And in `RabbitHoleReceipt.sol`
```diff
-166:          IQuest questContract = IQuest(questAddress);
+166:          Quest questContract = Quest(questAddress);
167:
168:          bool claimed = questContract.isClaimed(tokenId_);
-169:         uint rewardAmount = questContract.getRewardAmount();
+169:         uint rewardAmount = questContract.rewardAmountInWeiOrTokenId();
-170:         address rewardAddress = questContract.getRewardToken();
+170:         address rewardAddress = questContract.rewardToken();
```

Note that you must also remove the two getters from `IQuest()`


- Gas costs before:


| Contract            | Method            | Min    | Max    | Avg    |
|---------------------|-------------------|--------|--------|--------|
| QuestFactory        | Deployment        |        |        |5313516 |
|---------------------|-------------------|--------|--------|--------|
| QuestFactory        | createQuest       |1365913 |1508398 |1437156 |


- Gas costs after:

| Contract            | Method            | Min    | Max    | Avg    |
|---------------------|-------------------|--------|--------|--------|
| QuestFactory        | Deployment        |        |        |5266581 |
|---------------------|-------------------|--------|--------|--------|
| QuestFactory        | createQuest       |1346235 |1488720 |1417478 |


This saves on average:
- `46,935` gas upon deployment for `QuestFactory.sol`
- `19,678` gas on every `QuestFactory.createQuest` call

# `timestampForTokenId` can be removed

`timestampForTokenId` is used when minting a receipt in `mint()` to save the timestamp at which the token was minted.
This information can also be retrieved by checking the timestamp of the block in which the event `Transfer(address(0), tokenId, ..)` (emitted upon minting) was included in.
Given that `timestampForTokenId` is not used anywhere else, consider removing it to save gas upon deployment and on calls to `mintReceipt()`.


## Instances

### RabbitHoleReceipt.sol

```solidity
35:     mapping(uint => uint) public timestampForTokenId;
```

## Tools Used

Manual Analysis, Hardhat

## Mitigation

```diff
-35:     mapping(uint => uint) public timestampForTokenId;
...
104:         questIdForTokenId[newTokenID] = questId_;
-105:         timestampForTokenId[newTokenID] = block.timestamp;
106:         _safeMint(to_, newTokenID);
```


- Gas costs before:

| Contract            | Method            | Min    | Max    | Avg    |
|---------------------|-------------------|--------|--------|--------|
| RabbitHoleReceipt   | Deployment        |        |        |2775990 |
|---------------------|-------------------|--------|--------|--------|
| QuestFactory        | mintReceipt       |276604  | 288104 |282354  |


- Gas costs after:

| Contract            | Method            | Min    | Max    | Avg    |
|---------------------|-------------------|--------|--------|--------|
| RabbitHoleReceipt   | Deployment        |        |        |2759180 |
|---------------------|-------------------|--------|--------|--------|
| QuestFactory        | mintReceipt       |254430  | 265930 |260180  |


This saves on average:
- `16,810` gas upon deployment for `RabbitHoleReceipt`
- `22,174` gas on every `QuestFactory.mintReceipt` call

# `setQuestFactory` and `setMinter` can be combined in one function

As per `QuestFactory.mintReceipt` comments:

```solidity
215: /// @dev mint a RabbitHole Receipt. Note: this contract must be set as Minter on the receipt contract
```
This means that when the owner calls `RabbitHoleReceipt.setQuestFactory`, they must follow it with a call to `RabbitHoleReceipt.setMinter`.
Given that the base number of gas needed for any transaction is 21,000 gas, combining these two setters into one can save `21,000` gas every time the owner need to change the `QuestFactory` address.



## Instances

### RabbitHoleReceipt.sol

```solidity
76:     /// @dev set the quest factory contract
77:     /// @param questFactory_ the address of the quest factory contract
78:     function setQuestFactory(address questFactory_) public onlyOwner {
79:         QuestFactoryContract = IQuestFactory(questFactory_);
80:     }
81: 
82:     /// @dev set the minter address
83:     /// @param minterAddress_ the address of the minter
84:     function setMinterAddress(address minterAddress_) public onlyOwner {
85:         minterAddress = minterAddress_;
86:         emit MinterAddressSet(minterAddress_);
87:     }
```

## Tools Used

Manual Analysis,

## Mitigation

```diff
76:     /// @dev set the quest factory contract
77:     /// @param questFactory_ the address of the quest factory contract
78:     function setQuestFactory(address questFactory_) public onlyOwner {
79:         QuestFactoryContract = IQuestFactory(questFactory_);
+           minterAddress = questFactory_;
+           emit MinterAddressSet(questFactory_);
80:     }
81: 
-82:     /// @dev set the minter address
-83:     /// @param minterAddress_ the address of the minter
-84:     function setMinterAddress(address minterAddress_) public onlyOwner -{
-85:         minterAddress = minterAddress_;
-86:         emit MinterAddressSet(minterAddress_);
-87:     }
```


- Gas costs before:

| Contract            | Method            | Min    | Max    | Avg    |
|---------------------|-------------------|--------|--------|--------|
| RabbitHoleReceipt   | Deployment        |        |        |2775990 |



- Gas costs after:

| Contract            | Method            | Min    | Max    | Avg    |
|---------------------|-------------------|--------|--------|--------|
| RabbitHoleReceipt   | Deployment        |        |        |2757896 |



This saves on average:
- `18,096` gas upon deployment for `RabbitHoleReceipt`
- `21,000` gas on every `setQuestFactory` call and `setMinter` call (as they are combined in one)
