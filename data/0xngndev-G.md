### getRewardAmount and getRewardToken getters are not necessary
Public state variables generate their own getter functions. Both `rewardAmountInWeiOrTokenId` and `rewardToken` are public state variables. Therefore the getters are not needed and just add deployment costs.
You'll need to add `rewardAmountInWeiOrTokenId` and `rewardToken` to the interface.


### Quest struct can be packed into 2 slots instead of 4
`questAddress` takes 20 bytes, this means `totalParticipants` and `numberMinted` could fit in `12 bytes`. So, the struct could be packed like this:
```
struct Quest {
	mapping(address => bool) addressMint;
	address questAddress;
	uint48 totalParticipants;
	uint48 numberMinted;
}
```
uint48s have a capability of `2^48 - 1 = 281474976710655`. This means each quest can have a maximum of  `281474976710655` participants and a maximum of `281474976710655` receipts minted--which seems enough.
This will save two cold `SSTOREs` over the current implementation as well as `SSLOADs`.

### **getOwnedTokenIdsOfQuest can be refactored to make claim() cheaper**
The second for loop can be more efficient if we take advantage of the foundTokens variable set up at the beginning of the function in the first for loop. By using it to track the indexes we can ensure the first `foundTokens` indexes of `tokenIdsForQuest` are valid `tokenIds`, which allows us to remove the `if` in the second loop and reduce iterations by iterating only through `foundTokens` instead of `msgSenderBalance`.
According to hardhat gas reports this saves about 300 gas per call to `claim`.
It could also be interesting to explore the idea of implementing a `claim()` function that allows the caller to pass in `tokenIds` to reduce lookup costs. 
```
    function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens;

        for (uint i = 0; i < msgSenderBalance; ) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[foundTokens] = tokenId;
                ++foundTokens;
            }
            unchecked {
                ++i;
            }
        }
        
        uint[] memory filteredTokens = new uint[](foundTokens);
        
        for (uint i = 0; i < foundTokens; ) {
            filteredTokens[i] = tokenIdsForQuest[i];
            unchecked {
                ++i;
            }
        }
        return filteredTokens;

    }
```