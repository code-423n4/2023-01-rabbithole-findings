
# Missing check for address(0) when assigning values to address state variables on `RabbitHoleReceipt.initialize`, `RabbitHoleReceipt.setMinterAddress`, `RabbitHoleTickets.initialize`, `RabbitHoleTickets.setMinterAddress`

[`RabbitHoleReceipt.sol#L53`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L53)

```solidity
 function initialize(address receiptRenderer_, address royaltyRecipient_, address minterAddress_, uint royaltyFee_ )
       public initializer {
        ...
        minterAddress = minterAddress_;
        ...
 }
```

[`RabbitHoleReceipt.sol#L84`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L84)

```solidity
    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        ...
   }
```

[`RabbitHoleTickets.sol#L42`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L42)

```solidity
   function initialize(address ticketRenderer_, address royaltyRecipient_, address minterAddress_, uint royaltyFee_ ) 
       public initializer {
        ...
        minterAddress = minterAddress_;
        ...
 }
```
(`RabbitHoleTickets.sol#L74`){https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L74}

```solidity
    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        ...
   }
```
