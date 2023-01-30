G1. There is no need for the ``isClaimed()`` function:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104-L108

We can save gas by check ``claimedList`` directly:
```
uint len = tokens.length;
for (uint i; i < len; unchecked{i++}) {
            if (!claimedList) {
                redeemableTokenCount++;
            }
}
```

G2. ``onlyOwner`` is used  in both child method and parent method, eliminating one can save gas:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150

``onlyOwner`` is used for both ``withdrawRemainingTokens()`` and its super method which is also called in ``withdrawRemainingTokens()``:
```
 function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_);
```
One modifier ``onlyOwner`` can be eliminated to save gas here.

G3: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L185
To save gas here, just needs to assign ``royaltyRecipient`` to ``receiver``. There is no need for the stack variable ``royaltyPayment`` and the return statement. Further gas is saved by checking whether ``royaltyFee !=0``. 
```
function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount) {
        require(_exists(tokenId_), 'Nonexistent token');
 
        if(royaltyFee !=0) royaltyAmount = (salePrice_ * royaltyFee) / 10_000;
        receiver = royaltyRecipient;
    }

```

G4. https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109-L135
Much gas can be saved for ``getOwnedTokenIdsOfQuest() `` by doing two things: 1) Using only ONE for-loop to calculate the final ``filteredTokens``; 2) Revising the length of array ``filteredTokens`` directly using assembly.

```
 function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory  filteredTokens) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
         filteredTokens = new uint[](msgSenderBalance);
        uint foundTokens = 0;

        for (uint i; i < msgSenderBalance;) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);

            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                filteredTokens[i] = tokenId;
                foundTokens++;
            }

            unchecked{++i;}
        }
   
       // @save the size of the array
       assembly{
               mstore(filteredTokens, foundTokens)
      }
}
```

G5. https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L162
We can save gas here by not performing a zero address check for ``QuestFactoryContract``. Instead, the zero address check should be performed by less called functions: the constructor and ``setQuestFactory()``.  In this way, we know for sure that ``QuestFactoryContract != addresss(0)`` when ``tokenURI()`` is called. 

G6. https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96-L118
The ``claim()`` function calls ``_setClaimed(tokens)`` to iterate each token in ``tokens`` and blindly set it to true regardless of its truth value. This is a waste of gas. We can leverage the for-loop earlier (used to calculate ``redeemableTokenCount``) to save gas since we can avoid some blind writes here and another for-loop.

```
uint256 redeemableTokenCount = 0;
        for (uint i = 0; i < tokens.length; i++) {
            if (!claimedList[tokens[i]]) {
                redeemableTokenCount++;
                claimedList[tokens[i]] == true;   // @audit: only those false need to be changed to true
            }
        }
```

G7. Use uint for ``questId`` can save much gas. Currently, ``string`` is used for ``questId``. 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98

Suggestion:  using ``string`` for ``questId`` will waste much gas especially when it is used in mappings. Use ``uint`` instead and use OpenZeppelin's counter.sol for generating questId. Introduce ``questTitle`` to store the title information.
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Counters.sol

G8. It can be gas saving to eliminate the following checks (or later checks) since they will be checked again in later code. 
a) https://github.com/code-423n4/2023-01-numoen/blob/2ad9a73d793ea23a25a381faadc86ae0c8cb5913/src/core/Lendgine.sol#L87

