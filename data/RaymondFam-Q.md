## No storage gap for upgradeable contracts
Consider adding a storage gap at the end of an upgradeable contract, just in case it would entail some child contracts in the future. This would ensure no shifting down of storage in the inheritance chain.

```diff
+ uint256[50] private __gap;
```
Here are the contract instances entailed:

[File: QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)
[File: RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)

## `uint256` over `uint`
Across the codebase, there are numerous instances of uint, as opposed to uint256. In favor of explicitness, consider replacing all instances of `uint` with `uint256`.

Here are some of the instances entailed:

[File: QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)

```solidity
22:        uint totalParticipants;
23:        uint numberMinted;

31:    uint public questFee;
32:    uint public questIdCount;

193:    function getNumberMinted(string memory questId_) external view returns (uint) {

199:    function questInfo(string memory questId_) external view returns (address, uint, uint) {
```
## Unspecific compiler version pragma
For some source-units the compiler version pragma is very unspecific, i.e. ^0.8.15. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Non-compliant contract layout with Solidity's Style Guide
According to Solidity's Style Guide below:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the `view` and `pure` functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Consider adhering to the above guidelines for all contract instances entailed.

## Use a more recent version of solidity
The protocol adopts version 0.8.15 on writing contracts. For better security, it is best practice to use the latest Solidity version, 0.8.17.

Security fix list in the versions can be found in the link below:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

## Add a Constructor Initializer
As per Openzeppelin's recommendation:

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/6

The guidelines are now to prevent front-running of initialize() on an implementation contract, by adding an empty constructor with the initializer modifier. Hence, the implementation contract gets initialized atomically upon deployment.

This feature is readily incorporated in the Solidity Wizard since the UUPS vulnerability discovery. You would just need to check UPGRADEABILITY to have the following constructor instance below more equipped (just like it has been done so on [RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L38-L41)) as follows:

[File: QuestFactory.sol#L34-L35](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L34-L35)

```diff
    /// @custom:oz-upgrades-unsafe-allow constructor
-    constructor() initializer {}
+    constructor() {
+        _disableInitializers();
+    }
```
## Lack of events for critical operations
Critical operations not triggering events will make it difficult to review the correct behavior of the deployed contracts. Users and blockchain monitoring systems will not be able to detect suspicious behaviors at ease without events. Consider adding events where appropriate for all critical operations for better support of off-chain logging API.

Here are the setter instances with missing events entailed:

[File: QuestFactory.sol#L157-L189](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L157-L189)

```solidity
159:    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

165:    function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {

172:    function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {

179:    function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

186:    function setQuestFee(uint256 questFee_) public onlyOwner {
```
## Ownership can be renounced
Inherited Openzeppelin’s OwnableUpgradeable.sol implements `renounceOwnership()`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

Here is the contract instance entailed:

[File: QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)

## Lines too long
Lines in source code are typically limited to 80 characters, but it’s reasonable to stretch beyond this limit when need be as monitor screens theses days are comparatively larger. Considering the files will most likely reside in GitHub that will have a scroll bar automatically kick in when the length is over 164 characters, all code lines and comments should be split when/before hitting this length. Keep line width to max 120 characters for better readability where possible.

Here are some of the instances entailed:

[File: IQuestFactory.sol#L16](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L16)
[File: Erc20Quest.sol#L56-L57](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L56-L57)

## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

For instance, it will be of added values to the users and developers if:

- at least a minimalist NatSpec is provided on [IQuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol)
- @return is included in the function NatSpec of [`getOwnedTokenIdsOfQuest()`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L106-L112)

## Open TODOs
Open TODOs can point to architecture or programming issues that still need to be resolved. Consider resolving them before deploying.

Here are some of the instances entailed:

[File: IQuest.sol#L4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4)

```solidity
// TODO clean this whole thing up
```
## Camel casing input parameter names
State variable names should be conventionally camel cased where possible.

Here are some of the instances entailed:

[File: RabbitHoleReceipt.sol#L35-L36](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L35-L36)

```solidity
    ReceiptRenderer public ReceiptRendererContract;
    IQuestFactory public QuestFactoryContract;
```
## Missing boundary check on `royaltyFee`
A sanity boundary check should be implemented on `royaltyFee` when calling `setRoyaltyFee()` just as it has been done so in `setQuestFee()`.

[File: RabbitHoleReceipt.sol#L90-L93](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90-L93)

```solidity
    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```
## Use `delete` to clear variables
`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address or a boolean to false. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at x.

The delete key better conveys the intention and is also more idiomatic.

For instance, the `a = false` instance below may be refactored as follows:

[File: Quest.sol#L64](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L64)

```diff
-        isPaused = false;
+        delete isPaused;
```
## Typo errors
[File: QuestFactory.sol#L176](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176)

```diff
-    /// @dev set or remave a contract address to be used as a reward
+    /// @dev set or remove a contract address to be used as a reward
```
## `questIdCount` should be initialized to `0`
`questIdCount` in QuestFactory.sol is initialized to 1 when there is no quest created. Although there is no accounting mess up entailed, it should be initialized to `0` to more accurately reflect the number of quests created when its public getter is accessed.

Consider removing it from `initialize()` since not assigning anything to `questIdCount` is as good as keeping its default value, i.e. `0`:

[File: QuestFactory.sol#L37-L50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37-L50)

```diff
    function initialize(
        address claimSignerAddress_,
        address rabbitholeReceiptContract_,
        address protocolFeeRecipient_
    ) public initializer {
        __Ownable_init();
        __AccessControl_init();
        grantDefaultAdminAndCreateQuestRole(msg.sender);
        claimSignerAddress = claimSignerAddress_;
        rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
        setProtocolFeeRecipient(protocolFeeRecipient_);
        setQuestFee(2_000);
-        questIdCount = 1;
    }
```
## Parameterized custom errors
Custom errors can be parameterized to better reflect their respective error messages if need be.

For instance, the custom error instance below may be refactored as follows:

[File: IQuestFactory.sol#L5](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L5)

```diff
-    error QuestIdUsed();
+    error QuestIdUsed(string memory questId_);
```
[File: QuestFactory.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70)

```diff
-        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
+        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed(questId_);
```
## Boundary check 
`questFee`, an immutable variable in Erc20Quest.sol, should have a boundary check just as it has been adopted in [`setQuestFee()`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187) of QuestFactory.sol.

[File: Erc20Quest.sol#L38](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L38)

```diff
+    error QuestFeeTooHigh();

+        if (questFee_ > 10_000) revert QuestFeeTooHigh();
        questFee = questFee_;
```
## Events associated with setter functions
Consider having events associated with setter functions emit both the new and old values instead of just the new value.

Here are some of the instances entailed:

[File: RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol)

```solidity
68:        emit RoyaltyFeeSet(royaltyFee_);

75:        emit MinterAddressSet(minterAddress_);
```
## Over distribution of on-chain actions
The design of the protocol seems to be overselling or over-distributing on-chain actions as can be seen from the if block below:

[File: QuestFactory.sol#L220](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220)

```solidity
        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
```
It is not sure whether or not the users are going to be compensated via a new Erc20Quest contract. If not, the amount of on-chain actions to be completed should not exceed `totalParticipants` to avoid these specific DOS grievances.
 
## Threshold on boundary checks
It is recommended adding a threshold when setting boundary limits to the two opposite ends. For instance, the third if block in the `constructor()` of Quest.sol could end up having `endTime_ = startTime_ + 1` and still bypass the revert in this edge instance.

Consider having the if block refactored by adding a threshold of 7 days or any other suitable period as follows:

```diff
        if (endTime_ <= block.timestamp) revert EndTimeInPast();
        if (startTime_ <= block.timestamp) revert StartTimeInPast();
-        if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
+        if (endTime_ <= startTime_ + 7 days) revert EndTimeLessThanOrEqualToStartTime(); 
```
## Erc1155Quest unincentivized to promote on-chain actions
Devoid of `protocolFee` and its associated protocol rewards, Erc1155Quest is a lot less likely to have all participants complete the on-chain actions due to zero incentives given to promote the campaign.

Consider implementing `maxProtocolRewards()`, `protocolFee()`, and `withdrawFee()` like it has been done so in Erc20Quest where deemed fit.