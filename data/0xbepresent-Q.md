1 - A malicious owner can increase the protocol fee if a quest creator enters with a high amount of rewards.
==

The [QuestFactory.sol::setQuestFee()](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186) can be incremented by malicious owner if some Quest creator put a high rewards amount.

Add the fee incremention behind a timelock in order to protect the quest creators.

```solidity
File: QuestFactory.sol
186:     function setQuestFee(uint256 questFee_) public onlyOwner {
187:         if (questFee_ > 10_000) revert QuestFeeTooHigh();
188:         questFee = questFee_;
189:     }
```

2 - Validate the ```totalParticipants``` amount more than zero.
==

The [totalParticiapants is not validated](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L65) in the quest creation. There is no point in creating a quest that has zero participants.