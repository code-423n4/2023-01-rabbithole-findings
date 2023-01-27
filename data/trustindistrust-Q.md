# Low Severity
Descriptions below

* `withdrawRemainingTokens()` can be called against a quest more than once
* `claim()` can risk gas exhaustion on large receipt claims due to multiple mandatory loops
* `CREATE_QUEST_ROLE` should subtract one from hash output to prevent theoretical known preimage attacks
* Do not allow setting 100% fees in `QuestFactory.setQuestFee()`
* `QuestFactory.mintReceipt()` should add checks for state of quests, specifically `isPaused` and `isStarted`
* Future projects given ownership of Quests or `CREATE_QUEST_ROLE` right could soft-rug by pausing in-progress Quest indefinitely

### `Erc20Quest.withdrawFee()` can be called against a quest more than once

```
function withdrawFee() public onlyAdminWithdrawAfterEnd {
    IERC20(rewardToken).safeTransfer(protocolFeeRecipient, protocolFee());
}
```

The `withdrawFee()` function does not update the state of the Quest in such a way as to track claims against a set of currently-redeemed receipts. As such, even if no new users call `claim()`, protocol fees can be withdrawn more than once.

In the event of future decentralization of the protocol, unscrupulous projects could use this to reclaim tokens and deny valid receipts.

This multiple-claim issue factors into the Medium finding concerning denial of valid receipts due to griefing by 3rd parties.

---

### `Quest.claim()` can risk gas exhaustion on large receipt claims due to multiple mandatory loops

```
function claim() public virtual onlyQuestActive {
    if (isPaused) revert QuestPaused(); 

    uint256[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

    if (tokens.length == 0) revert NoTokensToClaim(); 

    uint256 redeemableTokenCount = 0;
    for (uint256 i = 0; i < tokens.length; i++) {
        if (!isClaimed(tokens[i])) { 
            redeemableTokenCount++;
        }
    }
    if (redeemableTokenCount == 0) revert AlreadyClaimed(); 

    uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
    _setClaimed(tokens);
    _transferRewards(totalRedeemableRewards);
    redeemedTokens += redeemableTokenCount;

    emit Claimed(msg.sender, totalRedeemableRewards);
}
```

The function contains 4 separate loops over arrays that the user cannot influence (such as only submitting a subset of claims at once). Two loops are in the call to `rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest()`, the overt `for` loop in the middle, and the loop in `_setClaimed`. 

This risk is small, but could be lessened somewhat by performing the boolean update contained in `_setClaimed()` inside the `if` branch in the existing `for` loop.

Otherwise, consider allowing partial claims of receipts via array arguments, or detecting expensive claims bundles and only partially iterating over them. Since the logic already filters out claimed receipts, this would make the later loops smaller.

---

### `CREATE_QUEST_ROLE` should subtract one from hash output to prevent theoretical known-preimage attacks

It is common practice to subtract 1 from the output of `keccak256` hashes when those outputs are used as keys, such as role systems and storage slots. This is so that the 'preimage' (input value) of the hash is unknowable.

---

### Do not allow setting 100% fees in `QuestFactory.setQuestFee()`

While the contract does already contain a check to disallow setting the fee over 100%, it would be better from a risk perspective to disallow setting 100% fees at all. A reasonable fee maximum should be checked for instead.

---

### `QuestFactory.mintReceipt()` should add checks for state of quests, specifically `isPaused` and `isStarted`

The `ClaimsSigner`, which was out of scope for this contest, is in charge of ensuring that claims hashes are only created for Quests that have started, not pre-start or after the `endTime` value. 

However, from a robustness perspective it would be best to add not just `isStarted` checks, but also `isPaused` checks to the `QuestFactory.mintReceipt()`. 

The pause check is especially valuable, as it dampens the incentive for 3rd party projects from pausing their Quests while still allowing receipts to accumulate, allowing a 'soft-rug' attack.

---

### Future projects given ownership of Quests or the `CREATE_QUEST_ROLE` right could soft-rug by pausing in-progress Quest indefinitely

While RabbitHole at launch will be the sole owner and deployer of all Quests and thus assumes the centralization risks therein, a future 'decentralization' of the protocol increases some risks for users interacting with Quests.

Specifically, a protocol could engineer a quest to require a very desireable on-chain action, and decide on an amount of tokens that they would be willing to permanently relinquish in exchange.

They would then begin the Quest normally, wait a short time, and then pause the Quest. Since there is no `isPaused` checking on the `QuestFactory.mintReceipt()`, valid user interactions could continue to accumulate even after the pause. 

This cycle could continue until word got out that the Quest was malicious. Users might have lost tokens, gas, time, etc performing the Quest, without hope of reward. Obviously, the malicious protocol would be unable to re-claim all of their tokens via `withdrawRemainingTokens()` because of the deduction math, but they might get some fraction of them back.

At such a time that Rabbithole is not the sole Quests controller, these risks should be accounted for in any contract updates.

---

# Non-critical
Descriptions below

* Consider renaming `error TotalAmountExceedsBalance` to be more descriptive
* Consider renaming `error EndTimeInPast` and `StartTimeInPast` to be more accurate to the test conditions
* Consider renaming `error AlreadyClaimed` to better reflect revert state
* Dev comment on `Erc1155Quest._calculateRewards` is inaccurate
* `withdrawRemainingTokens` functions contain duplicated check via overlapping modifiers
* Use `""` instead of `"0x00"` for empty bytes to avoid triggering extra computation on transfers
* `Quest.start()` sets already `false` value to `false`
* Consider renaming `Quest.onlyAdminWithdrawAfterEnd()` to reflect its actual check
* Consider refactoring `Quest.onlyQuestActive()` to contain quest paused check as well as existing checks
* Re-order local state changes in `Quest.claim()`
* Strict equality check in `Quest.isClaimed()` is unnecessary
* mapping `timestampForTokenId` appears to be underused, consider omitting it or documenting its future purpose


### Consider renaming `error TotalAmountExceedsBalance` to be more accurate to the test conditions

```
function start() public override {
    if (IERC20(rewardToken).balanceOf(address(this)) < maxTotalRewards() + maxProtocolReward()) { revert TotalAmountExceedsBalance(); }
    super.start();
}
```

```
function start() public override {
    if (IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId) < totalParticipants) { revert TotalAmountExceedsBalance(); }
    super.start();
}
```

This error doesn't clearly express the condition that wasn't satisfied, namely that the contract is _underfunded_ compared to the maximum quantity of rewards anticipated in its constructor. It even seems to indicate the opposite, that too many tokens were sent instead. 

Consider renaming it to `QuestUnderFunded` or `InsufficientRewardTokensDeposited` or the like. Also consider passing the value the contract is short as a parameter of the error.

---

### Consider renaming `error EndTimeInPast` and `error StartTimeInPast` to be more accurate to the test conditions

`Quest.sol` constructor

```
constructor(
    address rewardTokenAddress_,
    uint256 endTime_,
    uint256 startTime_,
    uint256 totalParticipants_,
    uint256 rewardAmountInWeiOrTokenId_,
    string memory questId_,
    address receiptContractAddress_
) {
    if (endTime_ <= block.timestamp) revert EndTimeInPast(); 
    if (startTime_ <= block.timestamp) revert StartTimeInPast();
    if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
    endTime = endTime_;
    startTime = startTime_;
    rewardToken = rewardTokenAddress_;
    totalParticipants = totalParticipants_;
    rewardAmountInWeiOrTokenId = rewardAmountInWeiOrTokenId_;
    questId = questId_;
    rabbitHoleReceiptContract = RabbitHoleReceipt(receiptContractAddress_);
    redeemedTokens = 0;
}
```

The constructor contains sanity checks on start and end time values. However, the text of the errors indicate past checks, but the actual checks are past and _present_. This introduces ambiguity on whether or not the tests themselves are accurate to intention.

Consider variants on `EndTimeCannotBePastOrCurrentBlock` to more fully express the test conditions.

---

### Consider renaming `error AlreadyClaimed` to better reflect revert reason

From `Quest.claim()`
```
uint256 redeemableTokenCount = 0;
for (uint256 i = 0; i < tokens.length; i++) {
    if (!isClaimed(tokens[i])) { // @audit-ok
        redeemableTokenCount++;
    }
}

if (redeemableTokenCount == 0) revert AlreadyClaimed();
```

The intention of this block is clear, but the error could be clarified to better express the revert reason. Specifically, the more precise reason for the revert is that there aren't any unclaimed tickets remaining for that user, rather than they've 'already claimed'. Semanticaly, a user who claims, and later buys another Receipt and claims a second time has also 'already claimed' but would still claim successfully.

Consider renaming the error to `NoUnclaimedReceiptsRemaining` or similar to more accurately express the revert reason.

---

### Dev comment on `Erc1155Quest._calculateRewards` is inaccurate

```
/// @dev Returns the amount of rewards. Since an 1155 is just one token, this returns itself
/// @param redeemableTokenCount_ The amount of reward tokens that the user is eligible for
/// @return The amount of reward tokens that the user is eligible for
function _calculateRewards(uint256 redeemableTokenCount_) internal pure override returns (uint256) {
    return redeemableTokenCount_; 
}
```

This function, rather than 'returning [the ERC1155 token]`, instead returns whatever value was passed as an argument to the function. '[...] returns itself' implies this would be invariant across calls.

This function is called during `Quest.claim()` and will use a non-user-controlled value, so there's no particular risk of subversion. That said, the comment is confusing in relation to the actual implementation.

---

### `withdrawRemainingTokens` functions contain duplicated check via overlapping modifiers

Both implementations of `withdrawRemainingTokens` in `Erc20Quest` and `Erc1155Quest` apply the `onlyOwner` modifier. However, in both cases the first call inside the scope is to `super.withdrawRemainingTokens`.

```
function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```

The virtual parent applies the same modifier, as well as another modifier.

Consider either removing the modifier on the child function, or applying the modifiers to the child so that they are easily seen and understood to be present.

---

### Use `""` instead of `"0x00"` for empty bytes to avoid triggering extra computation on transfers

When tokens with hooks on transfer take in `bytes` arguments, there is often a length check so that 0 length bytes inputs are skipped. By using `"0x00"`, a non-0 `bytes` value is created which is semantically empty but would fail the 0-length check. This would increase the gas cost of processing such transfers. 

Consider changing the value to `""` instead, unless there is some other reason to retain the bytes value as-is.

---

### `Quest.start()` sets already `false` value to `false`

Due to default values for booleans, it isn't necessary to set `isPaused` to `false`. Since the paused state value has its own setter functions (`pause()` `unPause()`) there isn't a risk of a DoS/bind by removing this mutation.

---

### Consider renaming `Quest.onlyAdminWithdrawAfterEnd()` to reflect its actual check

This modifier's name suggests it combines an admin/owner check with a check for the state of the Quest.

However, the modifier only contains the `endTime` check. It does not act as an ACL. Consider renaming it to better reflect its implementation, or adjust the implementation to conform to the name.

---

### Consider refactoring `Quest.onlyQuestActive()` to contain quest paused check as well as existing checks

This modifier is applied to `Quest.claim()` :
```
modifier onlyQuestActive() {
    if (!hasStarted) revert NotStarted();
    if (block.timestamp < startTime) revert ClaimWindowNotStarted();
    _;
}
```

`claim()` first checks if the Quest is paused. This check could easily be integrated into the modifier to add clarity to the code.

---

### Re-order local state changes in `Quest.claim()`

from `Quest.claim()`
```
_setClaimed(tokens);
_transferRewards(totalRedeemableRewards);
// move this ⬇️ above that ⬆️
redeemedTokens += redeemableTokenCount;
```

While there is no re-entrancy risk identified in this function, it would be best practice to reorder the statements to apply checks, mutations, side-effects ordering. This places the side effect of value transfer at the bottom of the state change list.

---

### Strict equality check in `Quest.isClaimed()` is unnecessary

```
function isClaimed(uint256 tokenId_) public view returns (bool) {
    return claimedList[tokenId_] == true;
}
```

This function, consulting the value of a `mapping(uint256 => bool)` already returns the semantically correct result, without applying the additional equality check. 

Test suite completed successfully after removing the equality check.

---

### mapping `RabbitHoleReceipt.timestampForTokenId` appears to be underused, consider omitting it or documenting its future purpose

`RabbitHoleReceipt.timestampForTokenId` is set during the `RabbitHoleReceipt.mint()` function, but isn't referenced anywhere else in the codebase.

Consider omitting the mapping, or documenting its intended purpose (off-chain computation, etc).