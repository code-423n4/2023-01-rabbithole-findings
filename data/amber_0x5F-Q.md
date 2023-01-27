## Extra double check modifier

The function `withdrawRemainingTokens(address _to)` of Quest.sol already has onlyOwner() modifier to check caller, so that Erc20Quest.sol & Erc1155Quest.sol doest not need to use `super.` to check caller again, should remove the onlyOwner modifier from Erc20Quest.sol & Erc1155Quest.sol.

Erc1155Quest.sol: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54

Erc20Quest.sol: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81
