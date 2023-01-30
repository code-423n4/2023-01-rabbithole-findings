# Gas Report
## Finding Summary

1. [**Explicit true/false Checks**](#1-Explicit-truefalse-Checks)
2. [**uint256 Iterator Checked Each Iteration**](#2-uint256-Iterator-Checked-Each-Iteration)
3. [**x += y Used For State Variables**](#3-x--y-Used-For-State-Variables)
4. [**Unnecessary Variable Initialization**](#4-Unnecessary-Variable-Initialization) ([+1](https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480#gas-7-dont-initialize-variables-with-default-value))
5. [**Variables Can Be Made Immutable**](#5-Variables-Can-Be-Made-Immutable)
6. [**Unnecessary Cast**](#6-Unnecessary-Cast)

## 1. Explicit true/false Checks

Explicit `false` checks are more expensive than using `!`. The same goes for `true` checks.

*/contracts/QuestFactory.sol*

```solidity
73:	if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
221:	if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
**Suggested Change**
```solidity
73:	if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();
221:	if (quests[questId_].addressMinted[msg.sender]) revert AddressAlreadyMinted();
```

## 2. uint256 Iterator Checked Each Iteration

A `uint256` iterator will not overflow before the check variable overflows. Unchecking the iterator increment saves gas.

**Example**
```solidity
//From
for (uint256 i; i < len; ++i) {
	//Do Something
}
//To
for (uint256 i; i < len;) {
	//Do Something
	unchecked { ++i; }
}
```

*/contracts/RabbitHoleReceipt.sol*

```solidity
117:	for (uint i = 0; i < msgSenderBalance; i++) {
128:	for (uint i = 0; i < msgSenderBalance; i++) {
```

*/contracts/Quest.sol*

```solidity
70:	for (uint i = 0; i < tokenIds_.length; i++) {
104:	for (uint i = 0; i < tokens.length; i++) {
```

## 3. x += y Used For State Variables

When dealing with state variables `x = x + y` is cheaper than `x += y`.

*/contracts/Quest.sol*

```solidity
115:	redeemedTokens += redeemableTokenCount;
```
**Suggested Change**
```solidity
115:	redeemedTokens = redeemedTokens + redeemableTokenCount;
```

## 4. Unnecessary Variable Initialization

In [Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol) the `redeemedTokens` variable is set to zero in the constructor. `redeemedTokens` is a `uint256` which defaults to zero anyways. Consider removing [L45](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45) to save gas.

*/contracts/Quest.sol*

```solidity
45:	redeemedTokens = 0;
```

## 5. Variables Can Be Made Immutable

Variables that are only set in the constructor can be made `Immutable` to save gas.

*/contracts/Quest.sol*

```solidity
21:	string public questId;
```
**Suggested Change**
```solidity
21:	string public immutable questId;
```

## 6. Unnecessary Cast

In [RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol) `QuestFactoryContract` is checked for the zero address. In checking for the zero address, the zero address is cast to type `IQuestFactory`. Consider removing the cast to save gas.

```solidity
162:	require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');
```
**Suggested Change**
```solidity
162:	require(QuestFactoryContract != address(0), 'QuestFactory not set');
```