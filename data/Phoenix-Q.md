
# Missing check for address(0) when assigning values to address state variables on `RabbitHoleReceipt.initialize`

[`RabbitHoleReceipt.sol#L53`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L53)

```solidity
 function initialize(address receiptRenderer_, address royaltyRecipient_, address minterAddress_, uint royaltyFee_ )
       public initializer {
        ...
        minterAddress = minterAddress_;
        ...
 }
```
