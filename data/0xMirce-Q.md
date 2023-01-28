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



## TransferOwnership should be two step process

In all contracts it is using the `OwnableUpgradeable` contract. The main problem with that contract is that It’s possible that the `onlyOwner` role mistakenly transfers ownership to the wrong address, resulting in a loss of the `onlyOwner` role. Instead of using the `OwnableUpgradeable` suggestion is using the `Ownable2StepUpgradeable` contract, so when an account with the `onlyOwner` role transfer ownership to the other account, the other account (`pendingOwner`) must confirm ownership by calling `acceptOwnership`and then he will become the new owner. In that way, even if the account with the `onlyOwner` mistake when calling `transferOwnership` transaction, it is possible to propose again a new pending owner and correct that mistake.



## Missing `onlyNotStarted` modifier in `start()` function

In line https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50 it is missing `onlyNotStarted` modifier. The reason why this modifier is missing is a case that this function has a same behavior as the `unPause` function when quest starts (if the quest starts and the user calls the `pause` function, it is possible to unpause, and with `start` and with `unpause` functions)