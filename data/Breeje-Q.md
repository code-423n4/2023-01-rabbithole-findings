## QA Report

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | USE OF FLOATING PRAGMA | 10 |
| [L-2](#L-2) | ABI.ENCODEPACKED() SHOULD NOT BE USED WITH DYNAMIC TYPES WHEN PASSING THE RESULT TO A HASH FUNCTION SUCH AS KECCAK256() | 3 |
| [L-3](#L-3) | SINGLE-STEP PROCESS FOR CRITICAL OWNERSHIP TRANSFER/RENOUNCE IS RISKY | 6 |
| [L-4](#L-4) | POOR IMPLEMENTATION OF OOPS CONCEPT IN `Quest` CONTRACT | 1 |
| [L-5](#L-5) | MISSING CHECKS FOR `address(0)` WHEN ASSIGNING VALUES TO ADDRESS STATE VARIABLES | 6 |
| [L-6](#L-6) | ADMIN WON'T HAVE ACCESS TO GRANT OR REVOKE ANY NEWER ROLE IN `QuestFactory.sol` | 1 |
| [L-7](#L-7) | `__ERC721Enumerable_init()` NOT CALLED IN `RabbitHoleReceipt` | 1 |
| [L-8](#L-8) | NO CHECK ON `royaltyFee` TO BE LESS THAN OR EQUAL TO 10_000 | 2 |
| [NC-1](#NC-1) | UNUSED PARAMETER SHOULD BE REMOVED | 3 |


### [L-1] USE OF FLOATING PRAGMA

Impact: [swcregistry](https://swcregistry.io/docs/SWC-103)

All the 10 Smart Contracts uses Floating Pragma Solidity Version.

### [L-2] ABI.ENCODEPACKED() SHOULD NOT BE USED WITH DYNAMIC TYPES WHEN PASSING THE RESULT TO A HASH FUNCTION SUCH AS KECCAK256()

Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`).

*Instances (3):*
```solidity
File: QuestFactory.sol

72:     if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {

105:    if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {

222:    if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)

### [L-3] SINGLE-STEP PROCESS FOR CRITICAL OWNERSHIP TRANSFER/RENOUNCE IS RISKY

Given that `Quest`, `QuestFactory`, `Erc20Quest`, `Erc1155Quest`, `RabbitHoleReceipt` and `RabbitHoleTickets` are derived from Ownable or OwnableUpgradable, the ownership management of these contract defaults to Ownable’s `transferOwnership()` and `renounceOwnership()` methods which are not overridden here. Such critical address transfer/renouncing in one-step is very risky because it is irrecoverable from any mistakes.

## Recommendation

It is recommended to use [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) to mitigate this risk.
If not, It is highly recommended to atleast override `renounceOwnership()` method and disable that option forever.

### [L-4] POOR IMPLEMENTATION OF OOPS CONCEPT IN `Quest` CONTRACT

`_calculateRewards`, `_transferRewards` and `withdrawRemainingTokens` methods in `Quest` contract uses either revert or contains the empty block.

*Instance:*
```solidity
File: Quest.sol

122:      function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
123:          revert MustImplementInChild();
124:      }

129:      function _transferRewards(uint256 amount_) internal virtual {
130:          revert MustImplementInChild();
131:      }

150:      function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol)

As `Quest` contract is inherited by `Erc20Quest` and `Erc1155Quest`, this methods are overriden in those contracts.

According to Solidity Docs, 
>Contract need to be marked as abstract when at least one of their functions is not implemented.

Given that the `Quest` contract is not implementing `withdrawRemainingTokens` method and same goes for `_calculateRewards` and `_transferRewards` too, so it should be marked as `abstract` contract.

Checkout [this](https://docs.soliditylang.org/en/v0.6.2/contracts.html#abstract-contracts) reference.

### [L-5] MISSING CHECKS FOR `address(0)` WHEN ASSIGNING VALUES TO ADDRESS STATE VARIABLES

*Instances (6):*
```solidity
File: QuestFactory.sol

37:   function initialize(
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
50:   }

160:  claimSignerAddress = claimSignerAddress_;

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37-L50)

```solidity
File: Quest.sol

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
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26-L46)

### [L-6] ADMIN WON'T HAVE ACCESS TO GRANT OR REVOKE ANY NEWER ROLE IN `QuestFactory.sol`

In `QuestFactory.sol` contract, `grantDefaultAdminAndCreateQuestRole` method is used at the time of initialization to grant role of `DEFAULT_ADMIN_ROLE` and `CREATE_QUEST_ROLE`.

```solidity
File: QuestFactory.sol

152:    function grantDefaultAdminAndCreateQuestRole(address account_) internal {
153:        _grantRole(DEFAULT_ADMIN_ROLE, account_);
154:        _grantRole(CREATE_QUEST_ROLE, account_);
155:    }

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152-L155)

What this internally does is call `_grantRole` method in `AccessControlUpgradable` which sets the `_account` as "member".

```solidity
File: AccessControlUpgradable.sol

    function _grantRole(bytes32 role, address account) internal virtual {
        if (!hasRole(role, account)) {
            _roles[role].members[account] = true;
            emit RoleGranted(role, account, _msgSender());
        }
    }

```

Owner can still add or remove from `CREATE_QUEST_ROLE` because of `changeCreateQuestRole` method, But owner won't be able to grant role or revoke role for any other Role as those methods has `onlyRole(getRoleAdmin(role))` modifier.

##### Recommended Mitigation Step

It is recommended to use `_setRoleAdmin` call which will enable `DEFAULT_ADMIN_ROLE` to create more roles, Grant them or revoke them in Future.

### [L-7] `__ERC721Enumerable_init()` NOT CALLED IN `RabbitHoleReceipt`

*Instance (1): *
```solidity
File: contracts/RabbitHoleReceipt.sol

      function initialize(
          address receiptRenderer_,
          address royaltyRecipient_,
          address minterAddress_,
          uint royaltyFee_
      ) public initializer {
          __ERC721_init('RabbitHoleReceipt', 'RHR');
          __ERC721URIStorage_init();
          __Ownable_init();
          royaltyRecipient = royaltyRecipient_;
          minterAddress = minterAddress_;
          royaltyFee = royaltyFee_;
          ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
      }

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43-L56)

### [L-8] NO CHECK ON `royaltyFee` TO BE LESS THAN OR EQUAL TO 10_000

It is recommended to add a require check on `royaltyFee` to be below or equal to 10_000 before setting it.

*Instances (2):*
```solidity
File: RabbitHoleTickets.sol 

    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66-L69)

```solidity
File: RabbitHoleReceipt.sol

    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90-L93)

### [NC-1] UNUSED PARAMETER SHOULD BE REMOVED

*Instances (3):*
```solidity
File: Quest.sol

122:      function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {

129:      function _transferRewards(uint256 amount_) internal virtual {

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L110)

```solidity
File: RabbitHoleTickets.sol

110:      uint256 tokenId_,

```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L110)