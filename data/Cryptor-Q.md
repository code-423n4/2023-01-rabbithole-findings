## _calcuateRewards is misleading

# Lines of code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L45-L50


# Proof of Concept
Consider the comments made above the _calcluateReward function
```
/// @dev Returns the amount of rewards. Since an 1155 is just one token, this returns itself
    /// @param redeemableTokenCount_ The amount of reward tokens that the user is eligible for
    /// @return The amount of reward tokens that the user is eligible for
```
According to the comments, the ERC1155 calculation is due to ERC1155 being a single token. However, ERC1155 is semi fungible, meaning that it can be configured to be one token or many tokens, making the below formula misleading

```
function _calculateRewards(uint256 redeemableTokenCount_) internal pure override returns (uint256) {
        return redeemableTokenCount_;
    }

```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Rename the variable to tokenId for accuracy