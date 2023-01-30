## TODO
Open TODO can point to an architecture or programming issue needing to be resolved. 

Here is a specific instance found.

[IQuest.sol#L4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4)

Suggested fix:

It is recommended resolving them before deploying.

## DOUBLE CALL ON ONLYOWNER
In Erc20Quest.sol and Erc1155Quest.sol, calling `withdrawRemainingTokens()` will end up having the modifier `onlyOwner()` invoked twice, i.e. first in the function visibility itself, and then in `super.withdrawRemainingTokens()` again.

[Erc20Quest.sol#L81-L82](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L82)
[Erc1155Quest.sol#L54-L55](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54-L55)

```
    function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_);
        ...
```
[Quest.sol#L150](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150)

```
    function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```
Suggested fix:

Remove `onlyOwner` from the child function visibility.

## COMMENT AND CODE LINE LENGTH
Code lines are typically limited to 80 characters, but may be stretched beyond this limit as monitor screens theses days are comparatively larger. Considering the files will most likely reside in GitHub that will have a scroll bar automatically kick in when the length is over 164 characters, all code lines and comments should be split before hitting this length. 

Here are the instances found.

[IQuestFactory.sol#L16](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L16)
[Erc20Quest.sol#L56-L57](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L56-L57)

Suggested fix:

Try limiting the length of comments and/or code lines to 80 - 100 characters long for readability sake.

## ON-CHAIN ACTIONS OVERSOLD
On-chain actions seems to be more than the total participants as is evidenced in the code lines below. This could lead to users completing their on-chain tasks but not being rewarded when `quests[questId_].totalParticipants` is hit.

[QuestFactory.sol#L219-L229](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L229)

```
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
        ...
```
Suggested fix:

Limit the amount of on-chain actions to `totalParticipants` or document clearly whether or not excess participants are going to be able to mint their receipts via a different `questId`.

## SPELLING MISTAKES
[QuestFactory.sol#L176](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176)

```
    @audit remave
    /// @dev set or remave a contract address to be used as a reward
```
## FUNCTIONS OF SIMILAR LOGIC
In Quest.sol, `pause()` and `unpause()` share similar code logic.

[Quest.sol#L55-L65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L55-L65)

```
    function pause() public onlyOwner onlyStarted {
        isPaused = true;
    }

    function unPause() public onlyOwner onlyStarted {
        isPaused = false;
    }
```
Suggested fix:

It is recommended combining them into 1 function that could toggle between `true` and `false`.

## CAMEL CASE
The following instances are named with a capital prefix.

[RabbitHoleReceipt.sol#L35-L36](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L35-L36)

```
    ReceiptRenderer public ReceiptRendererContract;
    IQuestFactory public QuestFactoryContract;
```
Suggested fix:

It is recommended adopting camel case when naming these public variables.

## STORAGE GAP FOR UPGRADEABLE CONTRACTS
Consider adding a storage gap at the end of each upgradeable contract. In the event some contracts needed to inherit from them, there would not be an issue shifting down of storage in the inheritance chain. Generally, storage gaps are a novel way of reserving storage slots in a base contract, allowing future versions of that contract to use up those slots without affecting the storage layout of child contracts. If not, the variable in the child contract might be overridden whenever new variables are added to it. This storage collision could have unintended and vulnerable consequences to the child contracts.

Here are the 2 contract instances found.

[QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)
[RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)

Suggested fix:

It is recommended adding the following code block at the end of the upgradeable contract:

```
    /**
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
     */
    uint256[49] private __gap;
```
## RENOUNCEABLE OWNERSHIP
When inheriting from Openzeppelin’s OwnableUpgradeable.sol, `renounceOwnership()` is one of the callable functions included. This could pose a risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby denying access to any functionality that is only callable by the owner.

Here is 1 specific contract instance found.

[QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)

## BOOLEAN EXPRESSIONS TO BOOLEAN LITERALS COMPARISON
It is not expedient comparing a boolean value to a boolean literal that would incur the additional `ISZERO` opcode operation.

Here are the 2 instances found.

[QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73)

```
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
```
[QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221)

```
        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
Suggested fix:

Remove `== true` and replace `== false` with the prefix negation `!`.

## THE USE OF DELETE TO RESET VARIABLES
`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address or a boolean to false. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at x.

The delete key better conveys the intention and is also more idiomatic.

Here are 2 specific instances found.

[Quest.sol#L50-L65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50-L65)

```
51:        isPaused = false;

64:        isPaused = false;
```
Suggested fix:

```
51:        delete isPaused;

64:        delete isPaused;
```
## QUESTIDCOUNT IS OVER COUNTED BY 1
In QuestFactory.sol, `questIdCount` is assigned `1` when `initialize()` is called. This leads to over counting by 1 when `++questIdCount` is executed in `createQuest()`, in the midst of creating an [Erc20Quest](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L101) or an [ERC1155Quest.](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L132)

[QuestFactory.sol#L37-L50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37-L50)

```
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
        questIdCount = 1;
    }
```
Suggested fix:

Initialize `questIdCount` to `0`.

## MISSING PROTOCOL FEES AND REWARDS IN ERC1155QUEST
It is noted that the Erc1155Quest is missing protocol fees and rewards that are found in ERC20Quest. This could lead to the former a lot less popularly known since no one is going to put in adequate efforts promoting the on-chain actions for free.

Suggested fix:

It is recommended implementing `maxProtocolRewards()`, `protocolFee()`, `withdrawFee()` and all other missing functionalities that are found in Erc20Quest.

## USE MORE RECENT VERSIONS OF SOLIDITY
Lower versions like 0.8.15 are being used in the protocol contracts. For better security, it is best practice to use the latest Solidity version, 0.8.17.

Please visit the versions security fix list in the link below for detailed info:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

## SOLIDITY COMPILER OPTIMIZATIONS COULD BE PROBLEMATIC
```
hardhat.config.js:
  29  module.exports = {
  30:   solidity: {
  31:     compilers: [
  32:       {
  33:         version: "0.8.15",
  34:         settings: {
  35:           optimizer: {
  36:               enabled: true,
  37:               runs: 1000000
  38
            }
```
Description: Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

Recommendation: Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## CONTRACT LAYOUT ON FUNCTION WRITINGS COMPLIANCE WITH SOLIDITY'S STYLE GUIDE
As denoted in Solidity's Style Guide:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the view and pure functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Where possible, consider adhering to the above guidelines for all contract instances entailed.
 
## USE UINT256 INSTEAD OF UINT
QuestFactory.sol is found to be using `uint` numerously in its code base.

[QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)

Suggested fix: 

For explicitness reason, it is recommended replacing all instances of `uint` with `uint256`.

## MISSING EVENTS ON CRITICAL OPERATIONS
Critical operations not triggering events will make it difficult to review the correct behavior of the contracts. Users and blockchain monitoring systems will not be able to easily detect suspicious behaviors without events.

Here are some of the instances found.

[RabbitHoleTickets.sol#L52-L62](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L52-L62)

```
    /// @dev set the ticket renderer contract
    /// @param ticketRenderer_ the address of the ticket renderer contract
    function setTicketRenderer(address ticketRenderer_) public onlyOwner {
        TicketRendererContract = TicketRenderer(ticketRenderer_);
    }

    /// @dev set the royalty recipient
    /// @param royaltyRecipient_ the address of the royalty recipient
    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
        royaltyRecipient = royaltyRecipient_;
    }
```
## NEW AND OLD VALUES SHOULD BE EMITTED
It is recommended having events associated with setter functions emit both the new and old values instead of just the new value. 

Here are some of the instances found.

[RabbitHoleTickets.sol#L64-L76](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L64-L76)

```
    /// @dev set the royalty fee
    /// @param royaltyFee_ the royalty fee
    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }

    /// @dev set the minter address
    /// @param minterAddress_ the address of the minter
    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```
## COMPILER VERSION PRAGMA SPECIFICITY
Non-library contracts and interfaces should avoid using floating pragmas ^0.8.15. Doing this may be a security risk for the actual application implementation itself. For instance, a known vulnerable compiler version may accidentally be selected or a security tool might fallback to an older compiler version leading to checking a different EVM compilation that is ultimately deployed on the blockchain.