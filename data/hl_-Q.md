# Table of Contents

- [N-01] Lock pragmas to specific compiler version
- [N-02] Use `bytes.concat()` instead of `abi.encodePacked()`
- [N-03] Import exact functions
- [N-04] Critical events should be indexed 
- [N-05] Confusing variable name
- [N-06] Open Todos
- [N-07] Function style guide not adhered
- [L-01] Overall tests percentage not 100%
- [L-02] Use function `safeTransferOwnership` instead of `transferOwnership` 
- [L-03] Quest endTime not indicated to users
- [L-04] Missing check to ensure endTime is after startTime
- [L-05] Missing zero address check

## [N-01] Lock pragmas to specific compiler version

Contracts should be deployed with the same compiler version that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using for e.g. a new compiler version which is not thoroughly tested, hence introducing bugs that affect the contract system negatively. https://swcregistry.io/docs/SWC-103

For example:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2

## [N-02] Use `bytes.concat()` instead of `abi.encodePacked()`

`bytes.concat()` can be used instead of `abi.encodePacked()` since version 0.8.4 for appending bytes.

For example: 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72

## [N-03] Import exact functions

Noted that the exact functions are not imported in various contracts. It is best practice to import exact functions rather than just the whole contracts. 

For example: 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L4-L13

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L4-L9

## [N-04] Critical events should be indexed 

As best practice, critical events should be indexed. 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L8

## [N-05] Confusing variable name

In ERC20Quest.sol, the variable name "nonClaimableTokens" in line 85 could be confused with the variable name in line 84 "unclaimedTokens". 

Given "nonClaimableTokens" is in relation to tokens that cannot be claimed due to unsuccessful quest, it is recommended to rename it to something more distinct from "unclaimedTokens". For example, "FailedQuestTokens". 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L84-L85

## [N-06] Open Todos

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4

## [N-07] Function style guide not adhered

Per solidity's documentation style guide, ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

    constructor
    fallback function (if exists)
    external
    public
    internal
    private

For example (public comes after internal here where it should be the other way round): 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L58-L87


## [L-01] Overall tests percentage not 100%

Noted per the contest scoping details that the overall line coverage percentage provided by tests is 89%. 

As best practice, it should be 100%. 

## [L-02] Use function `safeTransferOwnership` instead of `transferOwnership` 

`transferOwnership` function is used in QuestFactory.sol

It is more secure to use a 2-stage ownership transfer i.e. `safeTransferOwnership`

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L100

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L131

## [L-03] Quest endTime not indicated to users

Noted quest endTime in Quest.sol is not clearly indicated to users. This means that user can still do the quest after it ends, but they do not get any rewards. This may grieve and frustrate users, which in turn affects the reputation and reliability of RabbitHole. 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96

## [L-04] Missing check to ensure endTime is after startTime

In QuestFactory.sol, there is a missing check to ensure that endTime happens after startTime. It is best practice to incorporate this check into the contract to mitigate instances of human errors. 

## [L-05] Missing zero address check

In QuestFactory.sol, zero address checks are missing. It is best practice to incorporate this check into the contract.