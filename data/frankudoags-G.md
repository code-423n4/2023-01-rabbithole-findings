https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L69-L73

since overflow of tokenId_s.length is unrealistic, wrap the increment of `i` in an unchecked block to save gas.
```
unchecked {
++i
}
```


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104-L108

since overflow of tokens.length is unrealistic, wrap the increment of `i` in an unchecked block to save gas.
```
unchecked {
++i
}
```

Also, store the tokens.length outside of the for loop as a variable to save gas


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L158-L163

Use custom errors instead of long error strings in require statements are error strings longer than 32 gas requires more gas

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109-L135
wrap all increments in unchecked as overflow is unrealistic to save gas



