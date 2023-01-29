### getRewardAmount and getRewardToken getters are not necessary
Public state variables generate their own getter functions. Both `rewardAmountInWeiOrTokenId` and `rewardToken` are public state variables. Therefore the getters are not needed and just add deployment costs.
You'll need to add `rewardAmountInWeiOrTokenId` and `rewardToken` to the interface.

### getOwnedTokenIdsOfQuest can be refactored to make claim() cheaper
The if in the second for loop checks that `tokenId > 0` which is there because the length of the `tokenIdsForQuest` can be larger than the `foundTokens`. However, the first `foundTokens` elements of `tokensIdsForQuest` after the first for loop will be all populated with valid `tokenIds`. In other words, any index of `tokensIdsForQuest` over the number of `foundTokens` will be `0`. Therefore you can simply loop through `foundTokens`, and remove the if, and the filtered tracker.
This will cheapen `claim` as it reduces the number of iterations of the second for loop.
As an extra suggestion, it may be worth considering adding another `claim` function that allows the user to pass `tokenIds`. A function like that should save quite a bit of gas.
```
    function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens = 0;

        for (uint i = 0; i < msgSenderBalance; ) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
            unchecked {++i;}
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
