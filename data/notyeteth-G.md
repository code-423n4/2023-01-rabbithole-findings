Hi in the RabbitHoleReceipt.sol contract i've found a gas optisimation problem https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol


“for()” Loops and “unchecked{}” 

in the for loop for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }

        uint[] memory filteredTokens = new uint[](foundTokens);
        uint filterTokensIndexTracker = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            if (tokenIdsForQuest[i] > 0) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
        }
        return filteredTokens;
    }

In every case, using unchecked{ ++i; } is cheaper due to a few things:

It doesn’t use a default value of 0 for i, saves 3 gas per declaration

Using the iterator ++i ( ++i uses 5 gas less than i++ )

Using the unchecked{} block - as it is impossible for the iterator to overflow or underflow. A combination of this and ++i saves at least 25 gas, however can be up to 80 gas under certain conditions