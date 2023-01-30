### Gas Optimizations Report
| No. | Issue |
| --- | --- |
| 1 | [Using `uint256` instead of `bool` in mappings is more gas efficient [146 gas per instance]](#G-01-using-uint256-instead-of-bool-in-mappings-is-more-gas-efficient-146-gas-per-instance)|
| 2 | [++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#G-02-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops)|
| 3 | [Boolean Comparisions](#G-03-boolean-comparisions)|
| 4 | [Use assembly to check for `address(0)` [6 gas per instance]](#G-04-use-assembly-to-check-for-address0-6-gas-per-instance)|
| 5 | [Optimize names to save gas [22 gas per instance]](#G-05-optimize-names-to-save-gas-22-gas-per-instance)|
---
### [G-01] Using `uint256` instead of `bool` in mappings is more gas efficient [146 gas per instance]

---
**Description:**

OpenZeppelin uint256 and bool comparison: 

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled. The values being non-zero value makes deployment a bit more expensive.

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

**Recommendation:**

Use uint256(1) and uint256(2) instead of bool.

**Lines of Code:** 

- [Quest.sol#L24](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L24)
- [QuestFactory.sol#L20](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L20)
- [QuestFactory.sol#L30](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L30)

---
### [G-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

---
**Description:**

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

**Recommendation:**

Using Solidity's unchecked block saves the overflow checks.

**Proof Of Concept:**

<https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g011---unnecessary-checked-arithmetic-in-for-loop>

**Lines of Code:** 

- [Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70)
- [Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)
- [RabbitHoleReceipt.sol#L117](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117)
- [RabbitHoleReceipt.sol#L128](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128)

---
### [G-03] Boolean Comparisions

---
**Description:**

Comparing to a constant (`true` or `false`) is a bit more expensive than directly checking the returned boolean value.

**Recommendation:**

Using `if(value)` instead of `if(value) == true`

**Lines of Code:** 

- [Quest.sol#L136](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136)
- [QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221)

---
### [G-04] Use assembly to check for `address(0)` [6 gas per instance]

---
**Description:**



**Lines of Code:** 

- [QuestFactory.sol#L70](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70)
- [QuestFactory.sol#L166](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L166)

---
### [G-05] Optimize names to save gas [22 gas per instance]

---
**Description:**

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Recommendation:**

Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas. For example, the function IDs in the L1GraphTokenGateway.sol contract will be the most used; A lower method ID may be given.

**Proof Of Concept:**

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

**Lines of Code:** 

- [Erc1155Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol)
- [Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol)
- [Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol)
- [QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)
- [RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)
- [RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol)
- [ReceiptRenderer.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol)
- [TicketRenderer.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol)
- [IQuest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol)
- [IQuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol)