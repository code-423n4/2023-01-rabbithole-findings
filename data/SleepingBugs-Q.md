

# Low


# Critical address change should use two-step procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for [reference](https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure)

- [QuestFactory.sol#L159](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L159)
    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

- [QuestFactory.sol#L165](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L165)
    function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {

- [QuestFactory.sol#L172](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L172)
    function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {

- [QuestFactory.sol#L179](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L179)
    function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

- [QuestFactory.sol#L186](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L186)
    function setQuestFee(uint256 questFee_) public onlyOwner {

- [RabbitHoleReceipt.sol#L65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L65)
    function setReceiptRenderer(address receiptRenderer_) public onlyOwner {

- [RabbitHoleReceipt.sol#L71](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L71)
    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

- [RabbitHoleReceipt.sol#L77](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L77)
    function setQuestFactory(address questFactory_) public onlyOwner {

- [RabbitHoleReceipt.sol#L83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L83)
    function setMinterAddress(address minterAddress_) public onlyOwner {

- [RabbitHoleTickets.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L54)
    function setTicketRenderer(address ticketRenderer_) public onlyOwner {

- [RabbitHoleTickets.sol#L60](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L60)
    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

- [RabbitHoleTickets.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L73)
    function setMinterAddress(address minterAddress_) public onlyOwner {



## Recommendation

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

# Functions that change important state params should emit events

- [QuestFactory.sol#L159](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L159)
    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

- [QuestFactory.sol#L165](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L165)
    function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {

- [QuestFactory.sol#L172](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L172)
    function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {

- [QuestFactory.sol#L179](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L179)
    function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

- [QuestFactory.sol#L186](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L186)
    function setQuestFee(uint256 questFee_) public onlyOwner {

- [RabbitHoleReceipt.sol#L65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L65)
    function setReceiptRenderer(address receiptRenderer_) public onlyOwner {

- [RabbitHoleReceipt.sol#L71](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L71)
    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

- [RabbitHoleReceipt.sol#L77](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L77)
    function setQuestFactory(address questFactory_) public onlyOwner {

- [RabbitHoleTickets.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L54)
    function setTicketRenderer(address ticketRenderer_) public onlyOwner {

- [RabbitHoleTickets.sol#L60](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L60)
    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {



# `block.timestamp` used as time proxy 
## Summary

Risk of using `block.timestamp` for time should be considered. 
## Details

`block.timestamp` is not an ideal proxy for time because of issues with synchronization, miner manipulation and changing block times. 

This kind of issue may affect the code allowing or reverting the code before the expected deadline, modifying the normal functioning or reverting sometimes.
## References

SWC ID: 116

## Code Snippet 

- [RabbitHoleReceipt.sol#L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L102)
        timestampForTokenId[newTokenID] = block.timestamp;

- [Quest.sol#L35](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L35)
        if (endTime_ <= block.timestamp) revert EndTimeInPast();

- [Quest.sol#L36](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L36)
        if (startTime_ <= block.timestamp) revert StartTimeInPast();

- [Quest.sol#L77](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L77)
        if (block.timestamp < endTime) revert NoWithdrawDuringClaim();

- [Quest.sol#L90](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L90)
        if (block.timestamp < startTime) revert ClaimWindowNotStarted();

## Recommendation

- Consider the risk of using `block.timestamp` as time proxy and evaluate if block numbers can be used as an approximation for the application logic. Both have risks that need to be factored in. 
- Consider using an oracle for precision


# Destination recipient for assets may be `address(0)`

```solidity
IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);
```

[Erc20Quest.sol#L81](https://github.com/rabbitholegg/quest-protocol/blob/39bc02ddbdd306c8481fa655f899b8c8b8a0a1c2/contracts/Erc20Quest.sol#L81)

[Erc1155Quest.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/d6a07269ec55df8c95d13f5e06caca0f96c16587/contracts/Erc1155Quest.sol#L54)

Description

The recipient of a transfer may be `address(0)`, leading to lost assets.


# Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
## Impact 

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments" (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely. [See](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps).

For a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

## Proof of Concept

In the following context of the upgradeable contracts they are expected to use gaps for avoiding collision:

- `AccessControlUpgradeable`
- `ECDSAUpgradeable`
- `ERC1155BurnableUpgradeable`
- `ERC1155Upgradeable`
- `OwnableUpgradeable`
- `IERC2981Upgradeable`
- `Initializable`

However, none of these contracts contain storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Refer to the bottom part of this [article](https://docs.openzeppelin.com/contracts/4.x/upgradeable).

If a contract inheriting from a base contract contains additional variable, then the base contract cannot be upgraded to include any additional variable, because it would overwrite the variable declared in its child contract. This greatly limits contract upgradeability.

## Code Snippet

- [QuestFactory.sol#L15](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L15)

- [RabbitHoleReceipt.sol#L15](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L15)

- [RabbitHoleTickets.sol#L11](https://github.com/rabbitholegg/quest-protocol/blob/32a9b58f040442af4c7f459fe409e40af0e54a78/contracts/RabbitHoleTickets.sol#L11)


## Tools Used

Manual analysis

## Recommendation

Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. 

`uint256[50] private __gap;`

Reference OpenZeppelin [upgradeable contract templates](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable).






# Single-step process for critical ownership transfer/renounce is risky
## Impact

Given that `QuestFactory`, `RabbitHoleTickets`, `RabbitHoleReceipt`, `Quest` are derived from `Ownable` or `OwnableUpgradeable` the ownership management of this contract defaults to `Ownable` ’s `transferOwnership()` and `renounceOwnership()` methods which are not overridden here. 

Such critical address transfer/renouncing in one-step is very risky because it is irrecoverable from any mistakes

Scenario: If an incorrect address, e.g. for which the private key is not known, is used accidentally then it prevents the use of all the `onlyOwner()` functions forever, which includes the changing of various critical addresses and parameters. This use of incorrect address may not even be immediately apparent given that these functions are probably not used immediately. 

When noticed, due to a failing `onlyOwner()` function call, it will force the redeployment of these contracts and require appropriate changes and notifications for switching from the old to new address. This will diminish trust in the protocol and incur a significant reputational damage.

## Code Snippet

- [QuestFactory.sol#L16](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L16)
contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {

- [RabbitHoleReceipt.sol#L20](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L20)
    OwnableUpgradeable,

- [Quest.sol#L12](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L12)
contract Quest is Ownable, IQuest {

- [RabbitHoleTickets.sol#L14](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L14)
    OwnableUpgradeable,

## Recommendation

Recommend overriding the inherited methods to null functions and use separate functions for a two-step address change:
1. Approve a new address as a `pendingOwner`
2. A transaction from the `pendingOwner` address claims the pending ownership change.

This mitigates risk because if an incorrect address is used in step (1) then it can be fixed by re-approving the correct address. Only after a correct address is used in step (1) can step (2) happen and complete the address/ownership change.

Also, consider adding a time-delay for such sensitive actions. And at a minimum, use a multisig owner address and not an EOA.

# Missing checks for address(0x0) when assigning values to `address` state or `immutable` variables 

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480)

## Code Snippet

- [Quest.sol#L44](https://github.com/rabbitholegg/quest-protocol/blob/068d628f019e9469aecbf676370075c1f6c980fd/contracts/Quest.sol#L44)

- [QuestFactory.sol#L46](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L46)

- [RabbitHoleReceipt.sol#L55](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L55)

- [RabbitHoleTickets.sol#L44](https://github.com/rabbitholegg/quest-protocol/blob/32a9b58f040442af4c7f459fe409e40af0e54a78/contracts/RabbitHoleTickets.sol#L44)

## Recommendation

Check zero address before assigning or using it

# Missing checks for address(0x0) on some setters

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480)

## Code Snippet

- [QuestFactory.sol#L173](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L173)

- [RabbitHoleReceipt.sol#L66](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L66)

- [RabbitHoleReceipt.sol#L78](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L78)

- [RabbitHoleTickets.sol#L55](https://github.com/rabbitholegg/quest-protocol/blob/32a9b58f040442af4c7f459fe409e40af0e54a78/contracts/RabbitHoleTickets.sol#L55)

## Recommendation
Check zero address before assigning or using it




# Informational


## Solidity compiler optimizations can be problematic

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

It's likely there is a latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.
 
[Config](https://github.com/rabbitholegg/quest-protocol/blob/10e75638cd856a2bad442843c7d0c399dd4f0f2c/hardhat.config.ts#L29-L30)
          optimizer: {
            enabled: true,



## Constants should be defined rather than using magic numbers

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480)

- [QuestFactory.sol#L48](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L48)
        setQuestFee(2_000);

- [QuestFactory.sol#L187](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L187)
        if (questFee_ > 10_000) revert QuestFeeTooHigh();

- [ReceiptRenderer.sol#L60](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/ReceiptRenderer.sol#L60)
            generateAttribute('Reward Address', Strings.toHexString(uint160(rewardAddress_), 20)),

- [RabbitHoleReceipt.sol#L184](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L184)
        uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

- [RabbitHoleTickets.sol#L113](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L113)
        uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

- [Erc20Quest.sol#L53](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Erc20Quest.sol#L53)
        return (maxTotalRewards() * questFee) / 10_000;

- [Erc20Quest.sol#L97](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Erc20Quest.sol#L97)
        return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;


## Comparison of a boolean expression against a boolean value is redundant

There are instances where a boolean variable / function is checked against a boolean unnecessarily.

- Checks can be simplified from `variable == false` to `!variable`.
- Checks can be simplified from `variable == true` to `variable`.

- [QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L221)
        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

- [Quest.sol#L136](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L136)
        return claimedList[tokenId_] == true;

- [QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L73)
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();



## Open TODOs   

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

- [IQuest.sol#L4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/interfaces/IQuest.sol#L4)
// TODO clean this whole thing up


### Recommendation

Solve and remove `TODO`s 



### Recommendation

Use actual code






## Omissions in events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

- [RabbitHoleReceipt.sol#L85](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L85)
        emit MinterAddressSet(minterAddress_);

- [RabbitHoleReceipt.sol#L92](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L92)
        emit RoyaltyFeeSet(royaltyFee_);

- [RabbitHoleTickets.sol#L68](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L68)
        emit RoyaltyFeeSet(royaltyFee_);

- [RabbitHoleTickets.sol#L75](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L75)
        emit MinterAddressSet(minterAddress_);



## Missing 0 address check

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480)

0 address control should be done in these parts

- [QuestFactory.sol#L172-L174](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172-L174)

- [RabbitHoleReceipt.sol#L65-L67](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L65-L67)

- [RabbitHoleReceipt.sol#L77-L79](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L77-L79)

- [RabbitHoleTickets.sol#L54-L56](https://github.com/rabbitholegg/quest-protocol/blob/32a9b58f040442af4c7f459fe409e40af0e54a78/contracts/RabbitHoleTickets.sol#L54-L56)


## No same value input control

- [QuestFactory.sol#L159-L189](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159-L189)

- [RabbitHoleReceipt.sol#L65-L93](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L65-L93)

- [RabbitHoleTickets.sol#L54-L76](https://github.com/rabbitholegg/quest-protocol/blob/32a9b58f040442af4c7f459fe409e40af0e54a78/contracts/RabbitHoleTickets.sol#L54-L76)


### Recommendation

Add a check that same value it's not used. For example:

```diff
function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
        + if (claimSignerAddress_ == claimSignerAddress) revert ADDRESS_SAME();
        claimSignerAddress = claimSignerAddress_;
}
```


# Informationals


## Unused code

Code that is not used should be removed

- [Quest.sol#L129-L131](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L129-L131)

- [RabbitHoleReceipt.sol#L139-L141](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L139-L141)

- [Quest.sol#L122-L124](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L122-L124)


### Recommendation

Remove the code that is not used.

## Bad order of code

Clearness of the code is important for the readability and maintainability.
As Solidity guidelines [says](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) about order of the layout:
1. Type declarations
2. State variables
3. Events
4. Modifiers
5. Functions

[RabbitHoleReceipt.sol#L58](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L58)
    modifier onlyMinter() {

[Quest.sol#L76](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76)
    modifier onlyAdminWithdrawAfterEnd() {

[Quest.sol#L82](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L82)
    modifier onlyStarted() {

[Quest.sol#L88](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L88)
    modifier onlyQuestActive() {

[RabbitHoleTickets.sol#L47](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L47)
    modifier onlyMinter() {

## Missing Natspec 

NatSpec is missing for the following functions / constructor / modifiers:

[QuestFactory.sol#L37](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37)
    function initialize(

[RabbitHoleReceipt.sol#L43](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43)
    function initialize(

[Quest.sol#L74](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L74)


[RabbitHoleTickets.sol#L32](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32)
    function initialize(

[ReceiptRenderer.sol#L40](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40)
    function generateDataURI(

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L23)
    function isClaimed(uint256 tokenId_) external view returns (bool);

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L24)
    function getRewardAmount() external view returns (uint256);

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L25)
    function getRewardToken() external view returns (address);

[IQuestFactory.sol#L19](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L19)
    function questInfo(string memory questId_) external view returns (address, uint, uint);

## Undocumented parameters

In a some places, a parameter is missing in the documentation:

[QuestFactory.sol#L37](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37)
    function initialize(

[RabbitHoleReceipt.sol#L39](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L39)
    constructor() {

[RabbitHoleReceipt.sol#L43](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43)
    function initialize(

[RabbitHoleReceipt.sol#L158](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L158)
    function tokenURI(

[Quest.sol#L26](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26)
    constructor(

[Quest.sol#L74](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L74)


[Quest.sol#L120](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L120)
    /// @notice Calculate the amount of rewards 

[Quest.sol#L148](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L148)


[RabbitHoleTickets.sol#L32](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32)
    function initialize(

[RabbitHoleTickets.sol#L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102)
    function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {

[Erc1155Quest.sol#L13](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L13)
    constructor(

[ReceiptRenderer.sol#L40](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40)
    function generateDataURI(

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L23)
    function isClaimed(uint256 tokenId_) external view returns (bool);

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L24)
    function getRewardAmount() external view returns (uint256);

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L25)
    function getRewardToken() external view returns (address);

[IQuestFactory.sol#L19](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L19)
    function questInfo(string memory questId_) external view returns (address, uint, uint);

## Include return parameters in natspec comments

If Return parameters are declared, you must prefix them with ”/// @return”. [References](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html)

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the `@return` tag, they do incomplete analysis.

### Recommendation

Include return parameters in NatSpec comments

Recommendation Code Style:

```solidity
    /// @notice information about what a fooFighter function does
		/// @param fooParam what the fooParam represents
		/// @return what is the fooReturnValue returned by fooFighter
		function fooFighter(uint256 fooParam) public returns (uint256 fooReturnValue) {
      ...
    }
```

[QuestFactory.sol#L191](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L191)
    /// @dev return the number of minted receipts for a quest 

[QuestFactory.sol#L197](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L197)
    /// @dev return data in the quest struct for a questId 

[QuestFactory.sol#L207](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L207)
    /// @dev recover the signer from a hash and signature

[RabbitHoleReceipt.sol#L112](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L112)
    ) public view returns (uint[] memory) {

[RabbitHoleReceipt.sol#L181](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L181)
    ) external view override returns (address receiver, uint256 royaltyAmount) {

[RabbitHoleReceipt.sol#L190](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L190)
    function supportsInterface(

[Quest.sol#L74](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L74)


[Quest.sol#L133](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L133)
    /// @notice Checks if a Receipt token id has been used to claim a reward 

[Quest.sol#L140](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L140)
    function getRewardAmount() public view returns (uint256) {

[Quest.sol#L145](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L145)
    function getRewardToken() public view returns (address) {

[RabbitHoleTickets.sol#L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102)
    function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {

[RabbitHoleTickets.sol#L109](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L109)
    function royaltyInfo(

[RabbitHoleTickets.sol#L119](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L119)
    function supportsInterface(

[Erc20Quest.sol#L96](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L96)
    function protocolFee() public view returns (uint256) {

[ReceiptRenderer.sol#L40](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40)
    function generateDataURI(

[ReceiptRenderer.sol#L82](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L82)
    function generateAttribute(string memory key, string memory value) public pure returns (string memory) {

[ReceiptRenderer.sol#L100](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100)
    function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L23)
    function isClaimed(uint256 tokenId_) external view returns (bool);

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L24)
    function getRewardAmount() external view returns (uint256);

[IQuest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L25)
    function getRewardToken() external view returns (address);

[IQuestFactory.sol#L19](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L19)
    function questInfo(string memory questId_) external view returns (address, uint, uint);



## Miss leading name of modifier

Implementation doesn't only allow admins by itself to call it

```solidity
/// @notice Prevents reward withdrawal until the Quest has ended
modifier onlyAdminWithdrawAfterEnd() {
if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
_;
}
```

- [Quest.sol#L76](https://github.com/rabbitholegg/quest-protocol/blob/068d628f019e9469aecbf676370075c1f6c980fd/contracts/Quest.sol#L76)


## Missing initializer modifier on constructor

OpenZeppelin [recommends](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5) that the initializer modifier be applied to constructors in order to avoid potential griefs, [social engineering](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/4), or exploits. 

- [RabbitHoleReceipt.sol#L39](https://github.com/rabbitholegg/quest-protocol/blob/bd213e8629bb8587dd4bb35f3e9e8f8e42b40336/contracts/RabbitHoleReceipt.sol#L39)

- [RabbitHoleTickets.sol#L28](https://github.com/rabbitholegg/quest-protocol/blob/32a9b58f040442af4c7f459fe409e40af0e54a78/contracts/RabbitHoleTickets.sol#L28)