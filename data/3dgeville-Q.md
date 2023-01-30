First issue:

Function _burn in RabbitHoleReceipt.sol, the dev team should add a line to emit the burned tokens

Example: emit Burn(amount, tokenId_)

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L139-L141

Second issue:

Unlocked pragma:

I implore the devs in the project to lock the pragma version. A locked pragma insures that the codebase
is not accidentally deployed with the wrong compiler version with bugs.

Further reading on this subject:
see **[her](https://swcregistry.io/docs/SWC-103)**