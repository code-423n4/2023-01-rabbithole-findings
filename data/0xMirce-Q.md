## Optimize `endTime` and `startTime` checks 

With a small refactor https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L35:L37 , one check could be deleted.

Instead of 
```
	if (endTime_ <= block.timestamp) revert EndTimeInPast();
        if (startTime_ <= block.timestamp) revert StartTimeInPast();
        if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
```
it could be used
```
        if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime()
        if (startTime_ <= block.timestamp) revert StartTimeInPast();
``` 
because if `endTime` is greater than `startTime` and `startTime` is greater than `block.timestamp`, automatically `endTime` will be greater than `block.timestamp`.



## TransferOwnership should be two-step process

In all project contracts, it is using the `OwnableUpgradeable` contract. The main problem with that contract is that It’s possible that the `onlyOwner` role mistakenly transfers ownership to the wrong address, resulting in a loss of the `onlyOwner` role. Instead of using the `OwnableUpgradeable`, my suggestion is to use the `Ownable2StepUpgradeable` contract, so when an account with the `onlyOwner` role transfer ownership to the other account, the other account (`pendingOwner`) must confirm ownership by calling `acceptOwnership` and then he will become the new owner. In that way, even if the account with the `onlyOwner` mistake when calling the `transferOwnership` transaction, it is possible to propose a new pending owner again and correct that mistake.



## Missing `onlyNotStarted` modifier in `start()` function

In line https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50 it is missing `onlyNotStarted` modifier. The reason why this modifier is missing is a case that this function has the same behavior as the `unPause` function when the quest starts (if the quest starts and the user calls the `pause` function, it is possible to unpause, and with `start` and with `unpause` functions)


## It is possible to grant or revoke `CREATE_QUEST_ROLE` role and with only owner and with `DEFAULT_ADMIN_ROLE` role 

It is not so common that in the same contract exists Ownable and AccessControl access types. By default case in this contract, where `onlyOwner` and `DEFAULT_ADMIN_ROLE` are the same address, all will be without problems. 

But, if the address for one of `onlyOwner` or `DEFAULT_ADMIN_ROLE` is changed, then it would be possible to call grantRole and revokeRole with the `onlyOwner` account with function from line https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142 . In the other case, it will be possible with `DEFAULT_ADMIN_ROLE` directly call `QuestFactoryContract.grantRole()` or `QuestFactoryContract.revokeRole()`. This could cause potential divergence, and I suggest removing the function from the line https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142.