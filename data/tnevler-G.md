# Report
## Gas Optimizations ##
### [G-1]: The increment in for loop post condition can be made unchecked
**Context:**

1. ```for (uint i = 0; i < msgSenderBalance; i++) {``` [L117](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117) 
1. ```for (uint i = 0; i < msgSenderBalance; i++) {``` [L128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128) 
1. ```for (uint i = 0; i < tokenIds_.length; i++) {``` [L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70) 
1. ```for (uint i = 0; i < tokens.length; i++) {``` [L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104) 

**Description:**

[This](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked) can save 30-40 gas per loop iteration.

**Recommendation:**

Example how to fix. Change:
```
for (uint256 i = 0; i < orders.length; ++i) {
   // Do the thing
}
```

To:
```
for (uint256 i = 0; i < orders.length;) {
   // Do the thing
   unchecked { ++i; }
}
```
