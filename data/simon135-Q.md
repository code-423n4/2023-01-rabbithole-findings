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

