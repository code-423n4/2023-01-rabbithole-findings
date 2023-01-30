## [QA-01] **Checks Effects Interactions not properly followed in `claim` function.**

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96)

```solidity
function claim() public virtual onlyQuestActive {
        if (isPaused) revert QuestPaused();

        uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

        if (tokens.length == 0) revert NoTokensToClaim();

        uint256 redeemableTokenCount = 0;
        for (uint i = 0; i < tokens.length; i++) {
            if (!isClaimed(tokens[i])) {
                redeemableTokenCount++;
            }
        }

        if (redeemableTokenCount == 0) revert AlreadyClaimed();

        uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
        _setClaimed(tokens);
        _transferRewards(totalRedeemableRewards);
        redeemedTokens += redeemableTokenCount;

        emit Claimed(msg.sender, totalRedeemableRewards);
    }
```

In the function `claim` the rewards are tranfered before the change to `redeemedTokens` is made.