## Gas Optimizations
### [GAS-1] Use immutable values where possible
When a value is assigned to state in a constructor and is never modified afterwards, it is more gas efficient to make it immutable.

*1 Instance*
```solidity
File: Quest.sol

21:	string public questId;
```