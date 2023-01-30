# [L-1] Zero-Address check is missed
#### QuestFactory.sol
lines 179-180
lines 71-137 check rewardTokenAddress
lines 142-148
lines 159-161
lines 172-174
lines 179-181

# [L-2] transferReward might re-enter
#### Quest.sol
Claimed event should be emitted before calling setClaimed();

# [L-3] transferReward might re-enter
#### RabbitHoleReceipt.sol
line 54-62 create and emit events in setters

#[N-1] 
#### QuestFactory.sol
line 73 if (rewardAllowlist[rewardTokenAddress_] == false) change on if (!rewardAllowlist[rewardTokenAddress_])
line 221 if (quests[questId_].addressMinted[msg.sender] == true) change on if (quests[questId_].addressMinted[msg.sender])
