1.Erc1155Quest.getRewardAmount() Return error, the current implementation, each Receipt gets 1 Erc1155
It is recommended to override Quest.getRewardAmount() to return 1
```
contract Erc1155Quest is Quest, ERC1155Holder {
...
+   function getRewardAmount() public override view returns (uint256) {
+       return 1;
+   }
```

2.claim() does not follow the "Checks Effects Interactions" principle, resulting in external acquisition of redeemedTokens that may be inaccurate

```solidity
    function claim() public virtual onlyQuestActive {
...

        uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
        _setClaimed(tokens);
+       redeemedTokens += redeemableTokenCount;            
        _transferRewards(totalRedeemableRewards);
-       redeemedTokens += redeemableTokenCount;    
```

3.modifier "onlyAdminWithdrawAfterEnd" Naming problems, this does not limit must be admin suggested to modify the method named :onlyWithdrawAfterEnd
