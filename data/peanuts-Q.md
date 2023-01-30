### [L-01] Use safetransferownership instead of transferownership function.

transferOwnership function is used to change Ownership from QuestFactory.sol. Use a 2 structure transferOwnership (Ownable2Step) which is safer.
safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70-L75

### [L-02] Sanity checks in QuestFactory.sol

When creating a quest, make sure to check the 0 value of rewardTokenAddress_, totalParticipants, rewardAmountOrTokenId, contractType, questId etc. Also, sanity check for endTime > startTime_, endTime_ > block.timestamp and startTime_ > block.timestamp should come earlier.

    function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L68

### [L-03] Confusing pausing function application

In Quest.sol, the pause functions pauses the quest and the unPause function unpauses the quest. However, the pause and unpause function still works even after the endTime has passed. The claim() function can still be paused by the quest deployer after the endTime has passed, which is a centralization issue and shouldn't be the case. Pause and unpause should only work during the quest duration, and not before or after. Also, minting receipts can still be allowed when the quest is paused. 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57-L65

### [N-01] Non-library/interface files should use fixed compiler versions, not floating ones

Non-library/interface files should use fixed compiler versions, not floating ones

eg. 0.8.15 instead of ^0.8.15

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2

### [N-02] Use bytes.concat() instead of abi.encodePacked()

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled. Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();


        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();


            Erc20Quest newQuest = new Erc20Quest(

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70-L75

### [N-03] Update Import Usages for more readable and modern code

```
import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
```

eg. 

```
import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
```

### [N-04] Confusing variable name

In Erc20Quest.sol#withdrawRemainingTokens(), nonClaimableTokens should be renamed to unsuccessfulTokensToBeReclaimed. nonClaimableTokens sounds as though it is not claimable by anyone, which induces confusion, as the intention of the function is to just withdraw the excess tokens that have been deposited but participants were unable to successfully complete the quest.

    function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_);


        uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
        uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
        IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);
    }

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87

### [N-05] Improve test coverage

The test coverage rate of the project is 89%. Testing all functions is best practice in terms of security criteria. Due to its capacity, test coverage is expected to be 100%.

### [N-06] Function writing does not comply with the solidity style guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

All contracts, eg: 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L1-L151