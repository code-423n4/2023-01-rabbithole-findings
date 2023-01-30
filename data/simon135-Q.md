##### no check on address(0)
in most of the functions, there is no check for address(0)
##### owner can make `royaltyFee` high value and cause loss of funds for users trading tickets
```
   function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
```
##### constants should be defined rather than using magic numbers
```
        return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```
##### missing emit on a critical function
```
    questFee = questFee_;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L188
##### not good useability for users
 if a user doesn't want to claim a certain token, then they can't claim at all
Alice has 5  recipients tokens, the  4th token, she doesn't want to sell 
But in the `claim` function she has no choice but to claim the rewards 
https://github.com/rabbitholegg/quest-protocol/blob/068d628f019e9469aecbf676370075c1f6c980fd/contracts/Quest.sol#L99-L117
```solidity
       uint256[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
```
```solidity
        _setClaimed(tokens);

```
Have  the user give whatever  `tokenId` they want to claim 
