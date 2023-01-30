
## G-01 x += y costs more gas than x = x + y for state variables

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

_There is 1 instance of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
File: contracts/Quest.sol

115: redeemedTokens += redeemableTokenCount;
```

------

## G-02 Internal functions only called once can be inlined to save gas

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

_There are 3 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
File: contracts/Erc20Quest.sol

66: function _transferRewards(uint256 amount_) internal
74: function _calculateRewards(uint256 redeemableTokenCount_) internal
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

152: function grantDefaultAdminAndCreateQuestRole(address account_) internal {
```

-----

## G-03 Not using the named return variables when a function returns, wastes deployment gas

_There are 2 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

178: function royaltyInfo(
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

109: function royaltyInfo(
```

---

## G-04 Increments can be unchecked

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used. It saves **30-40 gas** per loop.

_There are 4 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
File: contracts/Quest.sol

70: for (uint i = 0; i < tokenIds_.length; i++) {
104: for (uint i = 0; i < tokens.length; i++) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

117: for (uint i = 0; i < msgSenderBalance; i++) {
128: for (uint i = 0; i < msgSenderBalance; i++) {
```

---

## G-05 Don't compare boolean expressions to boolean literals

Comparing to a constant (`true` or `false`) is a bit more expensive than directly checking the returned boolean value. I suggest using `if(!directValue)` instead of `if(directValue == false)`

`if (<x> == true)` => `if (<x>)`, `if (<x> == false)` => `if (!<x>)`

_There are 3 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
File: contracts/Quest.sol

136: return claimedList[tokenId_] == true;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

73: if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
221: if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```

-----

## G-06 Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

_There is 1 instance of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

17: bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```

----
