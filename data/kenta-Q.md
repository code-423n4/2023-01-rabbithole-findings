## 1: startTime_ and endTime_ must be checked.
startTime_ must be earlier than endTime_. A validation for it is missing in createQuest.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61

Add a new validation for the startTime_ and endTime_.
if (startTime_ >= endTime_) revert InvalidQuestDuration:

## 2: check the input account_ if it is an empty address in grantDefaultAdminAndCreateQuestRole. 

You need to deploy the contract again if the input account_ is an empty address because no account has the DEFAULT_ADMIN_ROLE and can not execute the grantRole.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152-L155

Add the validation if the input account_ is empty or not.

if (account_ == address(0)) revert AddressZeroNotAllowd();

## 3 fix the typo for the comment in setRewardAllowlistAddress

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176

Replace remave with remove
