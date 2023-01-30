### [Low-01] No checks on limit while setting ```royalityFee``` inside ```initialize()```
*Instances(2)*
```solidity
File:: RabbitHoleReceipt.sol
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L54
```
```solidity
File:: RabbitHoleTickets.sol
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L67
```


### [Low-02] Instead of ```_mint()``` use ```safeMint()```
*Instances(1)*
```solidity
File:: RabbitHoleTickets.sol
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L84
```

### [Low-03] Sending amount should be checked before transfering
even sending amount is 0 i.e ```nonClaimableTokens = 0```, transaction get successful i.e 0 amount sent
This is simply waste of gas,
so before transfering its good practice to confirm sending amount is non-zero
*Instances(1)*
```solidity
File:: Erc20Quest.sol
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L86
```


### [NC-01] Floating Pragma used
```solidity
File:: All those contract files in scope
```