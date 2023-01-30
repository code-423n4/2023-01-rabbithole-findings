#### [G-01] 
## `x+= y` costs more gas than `x = x + y` for state variables
#### Summary
Gas optimization can be made, when it is used x += y is more costly than x = x + y.   You can replace all  += or encounters to save gas. 
#### Github permalinks
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115

#### [G-02] 
## Incrementation optimization with ++I
#### Summary
Gas optimization can be made when it is used ++I then I++. It costs less gas to increment with prefix operation and when assigning the value.
#### Github permalinks
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L226
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L121
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L131
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L106


#### [G-03] 
## For loop increment I++ can be optimised with unchecked ++I
#### Summary
In the new versions of Solidity, it has a default overflow check. It can be saved gas at each iteration when unchecked is used.
Example:
```
for (uint256 i; i < arr.length; i++) {
    // ...
    }

```
Fix
```
 for (uint256 i; i < numIterations;) {
      // ...
      unchecked { ++i; }
    }

```
`no risk for overflow in the fixed version`
#### Github permalinks
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117-L123
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128-L135
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104-L108
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L69-L73 