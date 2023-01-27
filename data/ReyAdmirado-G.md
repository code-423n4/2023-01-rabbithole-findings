| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutable) |
| 2 | [expressions for constant values such as a call to keccak256(), should use immutable rather than constant](#2-expressions-for-constant-values-such-as-a-call-to-keccak256-should-use-immutable-rather-than-constant) |
| 3 | [state variables should be cached in stack variables rather than re-reading them from storage](#3-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage) |
| 4 | [Stack variable used as a cheaper cache for a state variable is only used once](#4-stack-variable-used-as-a-cheaper-cache-for-a-state-variable-is-only-used-once) |
| 5 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#5-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 6 | [not using the named return variables when a function returns, wastes deployment gas](#6-not-using-the-named-return-variables-when-a-function-returns-wastes-deployment-gas) |
| 7 | [can make the variable outside the loop to save gas](#7-can-make-the-variable-outside-the-loop-to-save-gas) |
| 8 | [`++x/x++` should be `unchecked{++x}/unchecked{x++}` when it is not possible for them to overflow](#8-xx-should-be-uncheckedxuncheckedx-when-it-is-not-possible-for-them-to-overflow) |
| 9 | [require() or revert() statements that check input arguments should be at the top of the function](#9-require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function) |
| 10 | [use custom errors rather than revert()/require() strings to save deployment gas](#10-use-custom-errors-rather-than-revertrequire-strings-to-save-deployment-gas) |
| 11 | [internal functions only called once can be inlined to save gas](#11-internal-functions-only-called-once-can-be-inlined-to-save-gas) |
| 12 | [public functions not called by the contract should be declared external instead](#12-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 13 | [Don’t compare boolean expressions to boolean literals](#13-dont-compare-boolean-expressions-to-boolean-literals) |
| 14 | [instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant](#14-instead-of-calculating-a-statevar-with-keccak256-every-time-the-contract-is-made-pre-calculate-them-before-and-only-give-the-result-to-a-constant) |
| 15 | [redundant assigning operation](#15-redundant-assigning-operation) |
| 16 | [can pre calculate parts of the code](#16-can-pre-calculate-parts-of-the-code) |

## 1. State variables only set in the constructor should be declared immutable.

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas).

`questId` is never changed after being constructed inside Quest Construct so it should be declared immutable
- [Quest.sol#L21](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L21)


## 2. expressions for constant values such as a call to keccak256(), should use immutable rather than constant

- [QuestFactory.sol#L](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L)


## 3. state variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`rewardToken` is being used twice and we can cache it to use one less storage read. used once in L85 and once in L86
- [Erc20Quest.sol#L85](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L85)

`rewardToken`is being used twice and we can cache it to use one less storage read. used once in L56 and once in L60
- [Erc1155Quest.sol#L56](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L56)

`rewardAmountInWeiOrTokenId` is being used twice and we can cache it to use one less storage read. used once in L59 and once in L60. make sure to cache it before L56
- [Erc1155Quest.sol#L59](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L59)

## 4. Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

`unclaimedTokens`
- [Erc20Quest.sol#L84](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L84)

`royaltyPayment`
- [RabbitHoleReceipt.sol#L184](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184)

`royaltyPayment`
- [RabbitHoleTickets.sol#L113](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113)

## 5. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas

- [Quest.sol#L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)


## 6. not using the named return variables when a function returns, wastes deployment gas

- [RabbitHoleReceipt.sol#L181](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L181)

- [RabbitHoleTickets.sol#L112](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L112)


## 7. can make the variable outside the loop to save gas

make the variable outside and only give the value to variable inside

`tokenId`
- [RabbitHoleReceipt.sol#L118](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L118)


## 8. `++x/x++` should be `unchecked{++x}/unchecked{x++}` when it is not possible for them to overflow

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this and save some gas but at the cost of some code readability.

- [Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70)
- [Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)
- [Quest.sol#L106](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)

- [QuestFactory.sol#L101](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L101)
- [QuestFactory.sol#L132](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L132)

- [RabbitHoleReceipt.sol#L117](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117)
- [RabbitHoleReceipt.sol#L121](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L121)
- [RabbitHoleReceipt.sol#L128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128)
- [RabbitHoleReceipt.sol#L131](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L131)


## 9. require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

change the position of if statements because the second one is cheaper
- [Quest.sol#L89-L90](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L89-L90)

change the position of if statements and sort them from cheapest to most expensive
- [QuestFactory.sol#L220-L223](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220-L223)


## 10. use custom errors rather than revert()/require() strings to save deployment gas

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

https://blog.soliditylang.org/2021/04/21/custom-errors/

- [RabbitHoleReceipt.sol#L161](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L161)
- [RabbitHoleReceipt.sol#L162](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L162)
- [RabbitHoleReceipt.sol#L182](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L182)


## 11. internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

`grantDefaultAdminAndCreateQuestRole` can be inlined
- [QuestFactory.sol#L152](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152)


## 12. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

- [Quest.sol#L50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)
- [Quest.sol#L57](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57)
- [Quest.sol#L63](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L63)
- [Quest.sol#L96](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96)
- [Quest.sol#L140](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L140)
- [Quest.sol#L145](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L145)
- [Quest.sol#L150](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150)

- [QuestFactory.sol#L61](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61)
- [QuestFactory.sol#L142](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142)
- [QuestFactory.sol#L159](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159)
- [QuestFactory.sol#L172](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172)
- [QuestFactory.sol#L179](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179)
- [QuestFactory.sol#L219](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219)

- [RabbitHoleReceipt.sol#L65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65)
- [RabbitHoleReceipt.sol#L71](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71)
- [RabbitHoleReceipt.sol#L77](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77)
- [RabbitHoleReceipt.sol#L83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83)
- [RabbitHoleReceipt.sol#L90](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90)
- [RabbitHoleReceipt.sol#L158](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L158)

- [RabbitHoleTickets.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54)
- [RabbitHoleTickets.sol#L60](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60)
- [RabbitHoleTickets.sol#L66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66)
- [RabbitHoleTickets.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73)
- [RabbitHoleTickets.sol#L92](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L92)
- [RabbitHoleTickets.sol#L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102)


## 13. Don’t compare boolean expressions to boolean literals

`if (<x> == true) => if (<x>)`
`if (<x> == false) => if (!<x>)`
`require(<x> == true) => require(<x>)`
`require(<x> == false) => require(!<x>)`


- [Quest.sol#L136](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136)

- [QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73)

- [QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221)


## 14. instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant

- [QuestFactory.sol#L](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L)

## 15. redundant assigning operation

`redeemedTokens` is already 0 no need to change it in constructor to 0. the whole line in redundant
- [Quest.sol#L45](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45)

## 16. can pre calculate parts of the code

`keccak256(abi.encodePacked('erc20'))` will always have the same answer so we can pre calculate the answer to it and use that in the code instead
- [QuestFactory.sol#L72](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72)

`keccak256(abi.encodePacked('erc1155'))`will always have the same answer so we can pre calculate the answer to it and use that in the code instead
- [QuestFactory.sol#L105](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105)
