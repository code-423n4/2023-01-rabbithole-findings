// @audit use ECDSAUpgradeable.toEthSignedMessageHash(hash_) for readability

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L211

---

Anyone can call `Erc20Quest.withdrawFee()` because modifier `onlyAdminWithdrawAfterEnd` does not check for admin. Either rename the modifier or add additional check.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L102
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76