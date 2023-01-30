# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 2 |
| 2  | Variables inside struct should be packed to save gas  | 1 |
| 3  | Use `unchecked` blocks to save gas  |  1 |
| 4  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  |  1 |
| 5  | Don't compare boolean expressions to boolean literals  | 2 |
| 6  | `require()` strings longer than 32 bytes cost extra gas | 4 |
| 7  | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops |  4  |


## Findings


### 1- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 2 instances of this issue :

```
File: contracts/BondNFT.sol

30      mapping(uint => string) public questIdForTokenId;
34      mapping(uint => uint) public timestampForTokenId;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct Receipt {
    uint256 questId;
    uint256 timestamp;
}
    
mapping(uint => Receipt) private _receiptForTokenId;
```

### 2- Variables inside `struct` should be packed to save gas :

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 1 instance of this issue:

File: QuestFactory.sol [Line 19-24](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L19-L24)

```
struct Quest {
    mapping(address => bool) addressMinted;
    address questAddress;
    uint totalParticipants;
    uint numberMinted;
}
```

As the `totalParticipants` and `numberMinted` parameters values can't really overflow 2^128 their values should be packed together and the struct should be modified as follow to save gas :

```
struct Quest {
    mapping(address => bool) addressMinted;
    address questAddress;
    uint128 totalParticipants;
    uint128 numberMinted;
}
```

### 3- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There is 1 instance of this issue:

File: QuestFactory.sol [Line 226](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L226)
```
quests[questId_].numberMinted++;
```

The above operation cannot overflow due to the check :

File: QuestFactory.sol [Line 220](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220)
```
if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
```

This check ensures that we always have `numberMinted <= totalParticipants` so the value of `numberMinted` can not overflow and the operation should be marked as `unchecked`. 


### 4- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 30 instances of this issue:

File: Quest.sol [Line 115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)
```
redeemedTokens += redeemableTokenCount;
```

### 5- Don't compare boolean expressions to boolean literals :

The following formulas should be used : `if (<x> == true)` => `if (<x>)`, `if (<x> == false)` => `if (!<x>)`

```
File: QuestFactory.sol

73       if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
221      if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```

### 6- `require()` strings longer than 32 bytes cost extra gas :

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas.

There is 1 instance of this issue :

File: RabbitHoleReceipt.sol [Line 161](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L161)
```
require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');
```

### 7- `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops :

There are 4 instances of this issue:
 
```
File: Quest.sol

70      for (uint i = 0; i < tokenIds_.length; i++)
104     for (uint i = 0; i < tokens.length; i++)

File: RabbitHoleReceipt.sol

117     for (uint i = 0; i < msgSenderBalance; i++)
128     for (uint i = 0; i < msgSenderBalance; i++)
```