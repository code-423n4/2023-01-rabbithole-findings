# GAS OPTIMIZATION REPORT
---
### Summary of optimizations
| Number | Issue details | Instances |
|---|---|:---:|
| [G-1](#G4) |`++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops | 4
| [G-2](#G6) |Using storage instead of memory for structs/arrays saves gas. | 2
| [G-3](#G12) |Internal functions only called once can be inlined to save gas. | 4
| [G-4](#G13) |`>=` costs less gas than `>`. | 3
| [G-5](#G15) |`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 1

*Total: 5 issues.*
*Note: The findings for issue with id 1 are different from the c4udit output.*

---
### Gas Optimizations


### <a id=G4>[G-1]</a> `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops

##### Description
In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck [cannot be made inline](https://github.com/ethereum/solidity/issues/10695). 
			Example for loop:

```Solidity
for (uint i = 0; i < length; i++) {
// do something that doesn't change the value of i
}
```

In this example, the for loop post condition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body `is 2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. You should manually do this by:

```Solidity
for (uint i = 0; i < length; i = unchecked_inc(i)) {
	// do something that doesn't change the value of i
}

function unchecked_inc(uint i) returns (uint) {
	unchecked {
		return i + 1;
	}
}
```
Or just:

```Solidity
for (uint i = 0; i < length;) {
	// do something that doesn't change the value of i
	unchecked { i++; 	}
}
```
Note that it’s important that the call to `unchecked_inc` is inlined. This is only possible for solidity versions starting from `0.8.2`.

Gas savings: roughly speaking this can save 30-40 gas per loop iteration. For lengthy loops, this can be significant!
(This is only relevant if you are using the default solidity checked arithmetic.)


##### Recommendation
Use the `unchecked` keyword

##### *Instances (4):*

File: [2023-01-rabbithole/quest-protocol/contracts/Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70 )
```solidity
70: for (uint i = 0; i < tokenIds_.length; i++) {

104: for (uint i = 0; i < tokens.length; i++) {

```
File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117 )
```solidity
117: for (uint i = 0; i < msgSenderBalance; i++) {
128: for (uint i = 0; i < msgSenderBalance; i++) {
```

### <a id=G6>[G-2]</a> Using storage instead of memory for structs/arrays saves gas.

##### Description
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.
Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct([src](https://ethereum.stackexchange.com/questions/128380/why-using-storage-keyword-instead-of-memory-cost-less-gas))

##### Recommendation
Use storage for `struct/array`

##### *Instances (2):*

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L114 )
```solidity
114: uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
125: uint[] memory filteredTokens = new uint[](foundTokens);
```
### <a id=G7>[G-3]</a> Empty blocks should be removed or emit something.

##### Description
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified
			```solidity
if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}
			```

##### Recommendation
Empty blocks should be removed or emit something (The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

##### *Instances (1):*
File: [2023-01-rabbithole/quest-protocol/contracts/Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150 )
```solidity
150: function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```

### <a id=G12>[G-3]</a> Internal functions only called once can be inlined to save gas.

##### Description
Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.
Check this out: [link](https://betterprogramming.pub/solidity-gas-optimizations-and-tricks-2bcee0f9f1f2)
Note: To sum up, when there is an internal function that is only called once, there is no point for that function to exist. 

##### Recommendation
Remove the function and make it inline if it is a simple function.

##### *Instances (4):*

File: [2023-01-rabbithole/quest-protocol/contracts/Erc1155Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L41 )
```solidity
41: function _transferRewards(uint256 amount_) internal override {

```
File: [2023-01-rabbithole/quest-protocol/contracts/Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L66 )
```solidity
66: function _transferRewards(uint256 amount_) internal override {

```
File: [2023-01-rabbithole/quest-protocol/contracts/Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L129 )
```solidity
129: function _transferRewards(uint256 amount_) internal virtual {

```
File: [2023-01-rabbithole/quest-protocol/contracts/QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152 )
```solidity
152: function grantDefaultAdminAndCreateQuestRole(address account_) internal {
```
### <a id=G13>[G-4]</a> `>=` costs less gas than `>`.

##### Description
The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which [saves 3 gas](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

##### Recommendation
Use `>=` instead if appropriate.

##### *Instances (3):*

File: [2023-01-rabbithole/quest-protocol/contracts/QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187 )
```solidity
187: if (questFee_ > 10_000) revert QuestFeeTooHigh();

220: if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();

```
File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L129 )
```solidity
129: if (tokenIdsForQuest[i] > 0) {
```

### <a id=G15>[G-5]</a> `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

##### Description
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

##### Recommendation
Use `<x> = <x> + <y>` instead.

##### *Instances (1):*

File: [2023-01-rabbithole/quest-protocol/contracts/Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115 )
```solidity
115: redeemedTokens += redeemableTokenCount;
```
