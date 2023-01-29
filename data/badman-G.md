Gas::gas optimization 

Change `if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();` to `if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();` can save small amount of gas, because of removing unnecessary opcode.

Link = https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73