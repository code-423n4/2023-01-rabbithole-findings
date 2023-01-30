- advised to have an enforced minimum duration between startTime and endTime e.g 24 Hours.
- in Quest.claim(), make sure to update state before external call to avoid any potential reentrancy attack that can manipulate redeemedTokens and therefore reduce the unclaimable tokens that can be withdrawn by withdrawRemainingTokens() 

```
        if (redeemableTokenCount == 0) revert AlreadyClaimed();

        uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
        _setClaimed(tokens);
        redeemedTokens += redeemableTokenCount; // moved to before external call
        _transferRewards(totalRedeemableRewards);

        emit Claimed(msg.sender, totalRedeemableRewards);
```