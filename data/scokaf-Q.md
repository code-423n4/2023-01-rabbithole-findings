# 1: FLOATING PRAGMA

Vulnerability details

### Impact

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.15, ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

For reference, see https://swcregistry.io/docs/SWC-103

## Proof of Concept


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L2 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L2 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Remove ^ in “pragma solidity ^0.8.15” and change it to “pragma solidity 0.8.15” to be consistent with the rest of the contracts.


# 2: USE A MORE RECENT VERSION OF SOLIDITY

Vulnerability details

## Context:

All contracts

## Description:

For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions; see https://github.com/ethereum/solidity/blob/develop/Changelog.md

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Old version of Solidity is used , newer version can be used (0.8.17)

# 3: USE CONSTANTS FOR NUMBERS

Vulnerability details

## Impact

In the Erc1155Quest.sol contract, numbers like 0x00 are used more than once. It is quite easy to make a mistake somewhere, also when comparing values.

## Proof of Concept

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L42 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L61 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Recommend defining constants for the numbers and values used throughout the code.

# 4: USE OF BLOCK.TIMESTAMP

Vulnerability details

### Context:

Block timestamps have been used historically for a number of purposes, including entropy for random numbers (see the Entropy Illusion for more information), locking funds for a set amount of time, and numerous state-changing time-dependent conditional statements. The ability of miners to slightly modify timestamps can be risky if block timestamps are used improperly in smart contracts.

Reference: https://swcregistry.io/docs/SWC-116 

## Proof of Concept

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L35 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L36 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L77 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L90  

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L102 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

1. Block timestamps shouldn't be utilized to create random numbers or entropy.

2. Use of trusted oracles
