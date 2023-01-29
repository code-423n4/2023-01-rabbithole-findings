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
