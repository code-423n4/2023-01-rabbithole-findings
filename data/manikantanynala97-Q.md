Precision Loss varies according to the token for the given Formula ie basically for USDC the decimal points are 6 and whereas for ETH it is 18 


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L95-L98


Check the nonClaimableTokens before transferring by calling the safe transfer function.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L85-L86