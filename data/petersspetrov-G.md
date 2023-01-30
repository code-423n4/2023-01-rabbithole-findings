[G-01] x = x - y; costs less gas than x -= y;, same for gathering You can replace all -= and += encounters to save gas.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115


[G-02]USING UNCHECKED BLOCKS TO SAVE GAS - INCREMENTS IN FOR LOOP CAN BE UNCHECKED ( SAVE 30-40 GAS PER LOOP ITERATION)
The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need 
to be checked for over/underflow because the variable will never reach the max number of uint256 
(will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117-L123

## Proof of Concept
File: contracts/RabbitHoleReceipt.sol
   for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }		
		
## Recommended Mitigation Steps:
for (uint i = 0; i < msgSenderBalance;) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
			unchecked {
               ++i;
           }
   
     }
		
		
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128	
## Proof of Concept
File: contracts/RabbitHoleReceipt.sol
   for (uint i = 0; i < msgSenderBalance; i++) {
            if (tokenIdsForQuest[i] > 0) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
        }

## Recommended Mitigation Steps:
for (uint i = 0; i < msgSenderBalance;) {
            if (tokenIdsForQuest[i] > 0) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
			unchecked {
               ++i;
           }
        }	