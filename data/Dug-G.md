# Gas optimizations

## Variable initialized with default value

Not sure why this instance was not caught by the C4audit output, but the `redeemedTokens = 0;` line in `Quest.sol` is unnecessary as the variable is already initialized to 0 by default.

```solidity
redeemedTokens = 0;
```

### Link to code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45

### Suggested improvement

Remove the line.


## Use boolean value directly instead of checking for equality to true

In `Quest.sol`, the `isClaimed` function returns `claimedList[tokenId_] == true`. It should just return the value directly instead of unnecessarily checking for equality to `true`.

```solidity
function isClaimed(uint256 tokenId_) public view returns (bool) {
    return claimedList[tokenId_] == true;
}
```

### Link to code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L135-L137

### Suggested improvement

```solidity
function isClaimed(uint256 tokenId_) public view returns (bool) {
    return claimedList[tokenId_];
}
```

## Reimplemented public getter functions

In `Quest.sol`, `getRewardAmount` and `getRewardToken` are just reimplementing the auto-generated getters for the public `rewardAmount` and `rewardToken` values. These functions could be removed.

```solidity
/// @dev Returns the reward amount
function getRewardAmount() public view returns (uint256) {
    return rewardAmountInWeiOrTokenId;
}

/// @dev Returns the reward token address
function getRewardToken() public view returns (address) {
    return rewardToken;
}
```

### Link to code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L139-L147

## Overriding with no changes in functionality

In `RabbitHoleReceipt.sol`, the `_burn` function is overriding the parent functions, but is not changing the functionality. This is unnecessary and could be removed.

```solidity
function _burn(uint256 tokenId_) internal override(ERC721Upgradeable, ERC721URIStorageUpgradeable) {
    super._burn(tokenId_);
}
```

### Link to code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L137-L141