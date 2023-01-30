# Save gas by create a compare function for string comparation

Seems kind of expensive for strings that have a different length, which could be most of the time.

https://fravoll.github.io/solidity-patterns/string_equality_comparison.html

```solidity
    function compare(string memory str1, string memory str2) public pure returns (bool) {
        return (bytes(str1).length == bytes(str2).length) && keccak256(abi.encodePacked(str1)) == keccak256(abi.encodePacked(str2));
    }
```

In `createQuest` we can rewrite

```solidity
        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20')))
        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155')))
```

with

```solidity
        if (compare(contractType_, 'erc20'))
        if (compare(contractType_, 'erc1155'))
```

# Returning the boolean value directly

```solidity
File: Quest.sol
135:     function isClaimed(uint256 tokenId_) public view returns (bool) {
136:         return claimedList[tokenId_] == true;
137:     }
```

the claimList itself is a boolean, just return the `claimedList[tokenId_]` rather than comparing it with true.
comparing with a `true` is unnecessary, removing it will save some gas and will return the same output.
furthermore, if needed just use the `claimedList` public variable directly to save some more gas.

# Save gas by preventing same action or action which doesn't change anything

for example:

```solidity
File: QuestFactory.sol
179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
180:         rewardAllowlist[rewardAddress_] = allowed_;
181:     }
```

we can add check, like `require(rewardAllowlist[rewardAddress_] != allowed_, "No changes");`

```solidity
    function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
        require(rewardAllowlist[rewardAddress_] != allowed_, "No changes");
        rewardAllowlist[rewardAddress_] = allowed_;
    }
```

# Integer overflow/underflow

Prior to Solidity 0.8.0 version, interger overflow and underflow checks were performed by using the SafeMath library. From Solidity 0.8.0 upward, the compiler does that check for us.

This extra check cost gas. If we know that the mathematical operations we will perform inside the contract will not overflow or underflow, we can tell the compiler not to check for overflow or underflow in the operation.

If we know that our mathematics will be safe we could safe a little gas by using unchecked.

```solidity
File: QuestFactory.sol
101:             ++questIdCount;
```

```solidity
File: Quest.sol
70:         for (uint i = 0; i < tokenIds_.length; i++) {
71:             claimedList[tokenIds_[i]] = true;
72:         }
```

the gas-saving pattern for the for-loops by using `unchecked`:

```solidity
for (uint256 i; i < tokenIds_.length; ) {
  claimedList[tokenIds_[i]] = true;
  unchecked {
    ++i;
  }
}
```

## Unnecessary loop

```solidity
File: RabbitHoleReceipt.sol
125:         uint[] memory filteredTokens = new uint[](foundTokens);
126:         uint filterTokensIndexTracker = 0;
127:
128:         for (uint i = 0; i < msgSenderBalance; i++) {
129:             if (tokenIdsForQuest[i] > 0) {
130:                 filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
131:                 filterTokensIndexTracker++;
132:             }
133:         }
```

the `if (tokenIdsForQuest[i] > 0)` will always return true because `tokenIdsForQuest[i] = tokenId;`, so it is unnecssary to have the `filteredTokens`. Removing it will save gas.

## Flatten internal function which being use only once

The `grantDefaultAdminAndCreateQuestRole` is only being used once, thus it's better to flatten this function.

```js
File: QuestFactory.sol
152:     function grantDefaultAdminAndCreateQuestRole(address account_) internal {
153:         _grantRole(DEFAULT_ADMIN_ROLE, account_);
154:         _grantRole(CREATE_QUEST_ROLE, account_);
155:     }
```
