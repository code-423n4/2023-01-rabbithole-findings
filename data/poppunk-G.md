```solidity
    /// @notice Allows user to claim the rewards entitled to them
    /// @dev User can claim based on the (unclaimed) number of tokens they own of the Quest
    function claim() public virtual onlyQuestActive {
        if (isPaused) revert QuestPaused();

        uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

        if (tokens.length == 0) revert NoTokensToClaim();

        uint256 redeemableTokenCount = 0;
        assembly {
            for {
                let i := 0
            } lt(i, mload(tokens)) {
                i := add(i, 1)
            } {
                let token := mload(add(tokens, mul(add(i, 1), 32)))
                let claimed := sload(add(token, 1))
                if iszero(claimed) {
                    redeemableTokenCount := add(redeemableTokenCount, 1)
                }
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

Replace current for loop in `claim` function with Yul to save gas. This will scale with the number of tokens returned from `getOwnedTokenIdsOfQuest` as it will bypass the array bounds checks for each loop iteration. The results can be drastic if there are a lot of tokens in a quest.