# Gas Optimization 1 

  In RabbitHoleReceipt.sol, getOwnedTokenIdsOfQuest, the foundTokens can be 0 so instead of looping from 0 to msgSenderBalance -1 (that can be cost quite a lot of gas if msgSenderBalance  is big) , should avoid that if the foundTokens is 0. 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L126-L133 
Propose to change 
```solidity
uint filterTokensIndexTracker = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            if (tokenIdsForQuest[i] > 0) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
        }

```
To: 
```solidity
if (foundTokens > 0) 
{ 
    uint filterTokensIndexTracker = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            if (tokenIdsForQuest[i] > 0) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
        }
} 


```

