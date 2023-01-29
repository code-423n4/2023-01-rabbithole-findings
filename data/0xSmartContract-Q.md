## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]| Protect your NFT from copying in fork|  |
|[L-02]| Missing Event for  initialize | 1 |
|[L-03]| Allows malleable SECP256K1 signatures| 1 |
|[L-04]| Lack of control to assign 0 values in the value assignments of critical state variables| 2 |
|[L-05]| Use ```safeTransferOwnership``` instead of ```transferOwnership``` function| 3 |
|[L-06]| Use `Ownable2StepUpgradeable` instead of ` OwnableUpgradeable ` contract|3  |
|[L-07]| Owner can renounce Ownership |  |
|[L-08]| Initialize() function can be called by anybody| 3 |
|[L-09]| Use `uint256` instead `uint`| 118 |
|[L-10]| Update codes to avoid Compile Errors| 6 |
|[L-11]| Front running attacks by the `onlyOwner` | 1 |
|[L-12]| Treatment of ERC-2981 for non-existent token royalty amount | 1 |
|[L-13]| Use re-entrancy guard in `RabbitHoleReceipt.mint()` function

Total 13 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [NC-01]|Add parameter to Event-Emit for critical function|1|
| [NC-02] |Insufficient coverage  |1|
| [NC-03] |NatSpec comments should be increased in contracts|  |
| [NC-04] |`Function writing` that does not comply with the `Solidity Style Guide` |  |
| [NC-05] |Add a timelock to critical functions | 1|
| [NC-06] |Tokens accidentally sent to the contract cannot be recovered|  |
| [NC-07] |Take advantage of Custom Error's return value property| 24 |
| [NC-08] |Repeated validation logic can be converted into a function modifier | 4 |
| [NC-09] |For modern and more readable code; update import usages | 23 |
| [NC-10] |Use a more recent version of Solidity| 10  |
| [NC-11] |For functions, follow Solidity standard naming conventions (internal function style rule)| 1 |
| [NC-12] |Floating pragma | 10 |
| [NC-13] |Project Upgrade and Stop Scenario should be|  |
| [NC-14] |Use SMTChecker|  |
| [NC-15] |Open TODO | 1 |
| [NC-16] |Lines are too long| 2 |
| [NC-17] |No same value input control| 13 |
| [NC-18] |Add NatSpec Mapping comment |  |
| [NC-19] |Include proxy contracts to Audit|  |
| [NC-20] |Omissions in Events| 2 |
| [NC-21] |Mark visibility of initialize(...) functions as ``external``| 3 |
| [NC-22] |Use of bytes.concat() instead of abi.encodePacked()| 7 |
| [NC-23] |Make the Test Context with Solidity|  |
| [NC-24] |Keccak Constant values should used to immutable rather than constant| 1|

Total 24 issues


### [L-01] Protect your NFT from copying in fork

Occasionally, forks can happen on different blockchains.
The project will operate on the Polygon network.Recently, a fork took place on the Ethereum network as well.

There may be forked versions of Blockchains, which could cause confusion and lead to scams as duplicated NFT assets enter the market, then it could result in duplication of non-fungible tokens (NFTs) and there's likely to be some level of confusion around which assets are 'official' or 'authentic.’

A forked Blockchains, chain will thus have duplicated deeds that point to the same tokenURI

About Replay Attack:
https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

The most important part here is NFT's tokenURI detail. If the update I suggested below is not done, Duplicate NFTs will appear as a result, potentially leading to confusion and scams.

### Tools Used
Manual Code Review


### Recommended Mitigation Steps

```diff
contracts/RabbitHoleReceipt.sol:
  157      /// @dev return the token uri, this delegates to the receipt renderer contract
  158:     function tokenURI(
  159:         uint tokenId_
  160:     ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {
+ 	 if (block.chainid != 137) revert wrongChain(); // Polygon Chain ID : 137
  161:         require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');
  162:         require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');
  163: 
  164:         string memory questId = questIdForTokenId[tokenId_];
  165:         (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
  166:         IQuest questContract = IQuest(questAddress);
  167: 
  168:         bool claimed = questContract.isClaimed(tokenId_);
  169:         uint rewardAmount = questContract.getRewardAmount();
  170:         address rewardAddress = questContract.getRewardToken();
  171: 
  172:         return ReceiptRendererContract.generateTokenURI(tokenId_, questId, totalParticipants, claimed, rewardAmount, rewardAddress);
  173:     }

```
### [L-02] Missing Event for  initialize

**Context:**
```solidity

contracts/QuestFactory.sol:
  36  
  37:     function initialize(
  38:         address claimSignerAddress_,
  39:         address rabbitholeReceiptContract_,
  40:         address protocolFeeRecipient_
  41:     ) public initializer {
  42:         __Ownable_init();
  43:         __AccessControl_init();
  44:         grantDefaultAdminAndCreateQuestRole(msg.sender);
  45:         claimSignerAddress = claimSignerAddress_;
  46:         rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
  47:         setProtocolFeeRecipient(protocolFeeRecipient_);
  48:         setQuestFee(2_000);
  49:         questIdCount = 1;
  50:     }

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit

### [L-03] Allows malleable SECP256K1 signatures

Here, the recover() method doesn't check the s range.

Homestead (EIP-2) added this limitation, however the precompile remained unaltered. The majority of libraries, including OpenZeppelin, do this check.
Since an order can only be confirmed once and its hash is saved, there doesn't seem to be a serious danger in existing use cases.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7201e6707f6631d9499a569f492870ebdd4133cf/contracts/utils/cryptography/ECDSA.sol#L138-L149


```solidity

contracts/QuestFactory.sol:
  209      /// @param signature_ The signature of the hash
  210:     function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
  211:         bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));
  212:         return ECDSAUpgradeable.recover(messageDigest, signature_);
  213:     }

```
### [L-04] Lack of control to assign 0 values in the value assignments of critical state variables


` royaltyFee ` and `setRoyaltyFee` variables are critical uint256 values. So should be check 0 value 
If the fees are 0 by mistake, even for a while, it will cause a loss to the project and owners

```solidity

contracts/RabbitHoleReceipt.sol:
  88      /// @dev set the royalty fee
  89:     /// @param royaltyFee_ the royalty fee
  90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  91:         royaltyFee = royaltyFee_;
  92:         emit RoyaltyFeeSet(royaltyFee_);
  93:     }
  94  

contracts/RabbitHoleTickets.sol:
  64      /// @dev set the royalty fee
  65:     /// @param royaltyFee_ the royalty fee
  66:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  67:         royaltyFee = royaltyFee_;
  68:         emit RoyaltyFeeSet(royaltyFee_);
  69:     }


```

### [L-05] Use ```safeTransferOwnership``` instead of ```transferOwnership``` function

**Context:**
```solidity
3 results - 1 file

contracts/Quest.sol:
   4: import {Ownable} from '@openzeppelin/contracts/access/Ownable.sol';
  12: contract Quest is Ownable, IQuest {

```
**Description:**
```transferOwnership``` function is used to change Ownership from ```Ownable.sol```.

Use a 2 structure transferOwnership which is safer. 
```safeTransferOwnership```,  use it is more secure due to 2-stage ownership transfer.

**Recommendation:**
Use ``Ownable2Step.sol``
[Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)

### [L-06] Use `Ownable2StepUpgradeable` instead of ` OwnableUpgradeable ` contract

**Context:**
```js
3 results - 3 files

contracts/QuestFactory.sol:
  16: contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {

contracts/RabbitHoleReceipt.sol:
  15: contract RabbitHoleReceipt is
  16:     Initializable,
  17:     ERC721Upgradeable,
  18:     ERC721EnumerableUpgradeable,
  19:     ERC721URIStorageUpgradeable,
  20:     OwnableUpgradeable,
  21:     IERC2981Upgradeable
  22: {

contracts/RabbitHoleTickets.sol:
  11: contract RabbitHoleTickets is
  12:     Initializable,
  13:     ERC1155Upgradeable,
  14:     OwnableUpgradeable,
  15:     ERC1155BurnableUpgradeable,
  16:     IERC2981Upgradeable
  17: {


```


**Description:**
```transferOwnership``` function is used to change Ownership from ```OwnableUpgradeable.sol```.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has  ` transferOwnership` function ,  use it is more secure due to 2-stage ownership transfer.

[Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)

### [L-07] Owner can renounce Ownership

**Context:**
[QuestFactory.sol#L9](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9)


**Description:**
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.


Also, if onlyOwner's Private Key is at risk of working, this function will be very dangerous.

` onlyOwner ` functions;
```solidity

contracts/QuestFactory.sol:
  142:     function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
  159:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
  165:     function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
  172:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
  179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
  186:     function setQuestFee(uint256 questFee_) public onlyOwner {
```

**Recommendation:**
We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### [L-08] initialize() function can be called by anybody

`initialize()` function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init` function.

Here is a definition of `initialize()` function.

```solidity

3 results 

contracts/QuestFactory.sol:
  36  
  37:     function initialize(
  38          address claimSignerAddress_,

contracts/RabbitHoleReceipt.sol:
  42  
  43:     function initialize(
  44          address receiptRenderer_,

contracts/RabbitHoleTickets.sol:
  31  
  32:     function initialize(
  33          address ticketRenderer_,

```


### Recommended Mitigation Steps

Add a control that makes `initialize()` only call the Deployer Contract or EOA;

```js
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }
```

### [L-09] Use `uint256` instead `uint`

Project use uint and uint256
 
Number of uses:
uint  = 118 results – 10 files
uint256 = 72 results - 10 files


Some developers prefer to use `uint256` because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example if doing ```bytes4(keccak('transfer(address, uint)’))```, you'll get a different method sig ID than ```bytes4(keccak('transfer(address, uint256)’))``` and smart contracts will only understand the latter when comparing method sig IDs



### [L-10] Update codes to avoid Compile Errors


```solidity

Warning: Unreachable code.
   --> contracts/Quest.sol:113:9:
    |
113 |         _setClaimed(tokens);
    |         ^^^^^^^^^^^^^^^^^^^


Warning: Unreachable code.
   --> contracts/Quest.sol:114:9:
    |
114 |         _transferRewards(totalRedeemableRewards);
    |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Unreachable code.
   --> contracts/Quest.sol:115:9:
    |
115 |         redeemedTokens += redeemableTokenCount;
    |         ^ (Relevant source part starts here and spans across multiple lines).


Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/Quest.sol:122:32:
    |
122 |     function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
    |                                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/Quest.sol:129:31:
    |
129 |     function _transferRewards(uint256 amount_) internal virtual {
    |                               ^^^^^^^^^^^^^^^


Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/RabbitHoleTickets.sol:110:9:
    |
110 |         uint256 tokenId_,
    |         ^^^^^^^^^^^^^^^^

```

### [L-11] Front running attacks by the `onlyOwner` 


Project has one  possible attack vectors by the `onlyOwner`:

With the function ` setQuestFee` the following is set: The quest fee value

When a user uses the platform expecting a low quest fee value , the owner can run the " setQuestFee " function up front and raise the price significantly. If the size is large enough, this can be a substantial amount of money.

```solidity
contracts/QuestFactory.sol:
  185      /// @param questFee_ The quest fee value
  186:     function setQuestFee(uint256 questFee_) public onlyOwner {
  187:         if (questFee_ > 10_000) revert QuestFeeTooHigh();
  188:         questFee = questFee_;
  189:     }
```


### Recommended Mitigation Steps

Use a timelock to avoid instant changes of the parameters.

### [L-12] Treatment of ERC-2981 for non-existent token royalty amount


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L178-L186


### Impact


if tokenId is non-existing  when require in ` RabbitHoleReceipt.royaltyInfo()` function  ,it will be revert


```solidity
contracts/RabbitHoleReceipt.sol:
  177      /// @param salePrice_ the sale price
  178:     function royaltyInfo(
  179:         uint256 tokenId_,
  180:         uint256 salePrice_
  181:     ) external view override returns (address receiver, uint256 royaltyAmount) {
  182:         require(_exists(tokenId_), 'Nonexistent token');
  183: 
  184:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
  185:         return (royaltyRecipient, royaltyPayment);
  186:     }

```

But if tokenId is non-existing  when require in `RabbitHoleTickets.royaltyInfo()` function  ,it will not be revert


```solidity
contracts/RabbitHoleTickets.sol:
  108      /// @param salePrice_ the sale price
  109:     function royaltyInfo(
  110:         uint256 tokenId_,
  111:         uint256 salePrice_
  112:     ) external view override returns (address receiver, uint256 royaltyAmount) {
  113:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
  114:         return (royaltyRecipient, royaltyPayment);
  115:     }

```


### Tools Used
Manuel Code Review


### Recommended Mitigation Steps
Add to require in `RabbitHoleTickets.royaltyInfo()` function;

```diff

contracts/RabbitHoleTickets.sol:
  108      /// @param salePrice_ the sale price
  109:     function royaltyInfo(
  110:         uint256 tokenId_,
  111:         uint256 salePrice_
  112:     ) external view override returns (address receiver, uint256 royaltyAmount) {
+                 require(_exists(tokenId_), 'Nonexistent token');
  113:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
  114:         return (royaltyRecipient, royaltyPayment);
  115:     }


```


### [L-13] Use re-entrancy guard in `RabbitHoleReceipt.mint()` function

Using the safemint pattern and following the check-effects-interaction pattern can help prevent some common security vulnerabilities in ERC721 contracts. Since the check-effects-interaction pattern is implemented in the `safeMint` function of the project, there does not appear to be a serious security problem.

However, it is still a good practice to apply a reentrancy guard as an additional measure of protection against potential reentrancy attacks.


```solidity

contracts/RabbitHoleReceipt.sol:
   97      /// @param questId_ the quest id
   98:     function mint(address to_, string memory questId_) public onlyMinter {
   99:         _tokenIds.increment();
  100:         uint newTokenID = _tokenIds.current();
  101:         questIdForTokenId[newTokenID] = questId_;
  102:         timestampForTokenId[newTokenID] = block.timestamp;
  103:         _safeMint(to_, newTokenID);
  104:     }

```



### Tools Used
Manuel Code Review



### Recommended Mitigation Steps
Use Openzeppelin or Solmate Re-Entrancy pattern
Here is a example of a re-entrancy guard

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract ReEntrancyGuard {
    bool internal locked;

    modifier noReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
    }
}
```

```diff

contracts/RabbitHoleReceipt.sol:
   97      /// @param questId_ the quest id
-  98:     function mint(address to_, string memory questId_) public onlyMinter {
+  98:     function mint(address to_, string memory questId_) public noReentrant onlyMinter {
   99:         _tokenIds.increment();
  100:         uint newTokenID = _tokenIds.current();
  101:         questIdForTokenId[newTokenID] = questId_;
  102:         timestampForTokenId[newTokenID] = block.timestamp;
  103:         _safeMint(to_, newTokenID);
  104:     }

```



### [N-01] Add parameter to Event-Emit for critical function

Some event-emit description hasn’t parameter. Add to parameter  for front-end website or client app , they can has that something has happened on the blockchain.


```solidity

 function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();

        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

            Erc20Quest newQuest = new Erc20Quest(
                rewardTokenAddress_,
                endTime_,
                startTime_,
                totalParticipants_,
                rewardAmountOrTokenId_,
                questId_,
                address(rabbitholeReceiptContract),
                questFee,
                protocolFeeRecipient
            );

            emit QuestCreated(
                msg.sender,
                address(newQuest),
                questId_,
                contractType_,
                rewardTokenAddress_,
                endTime_,
                startTime_,
                totalParticipants_,
                rewardAmountOrTokenId_
            );
            quests[questId_].questAddress = address(newQuest);
            quests[questId_].totalParticipants = totalParticipants_;
            newQuest.transferOwnership(msg.sender);
            ++questIdCount;
            return address(newQuest);
        }

        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
            if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest();

            Erc1155Quest newQuest = new Erc1155Quest(
                rewardTokenAddress_,
                endTime_,
                startTime_,
                totalParticipants_,
                rewardAmountOrTokenId_,
                questId_,
                address(rabbitholeReceiptContract)
            );

            emit QuestCreated(
                msg.sender,
                address(newQuest),
                questId_,
                contractType_,
                rewardTokenAddress_,
                endTime_,
                startTime_,
                totalParticipants_,
                rewardAmountOrTokenId_
            );
            quests[questId_].questAddress = address(newQuest);
            quests[questId_].totalParticipants = totalParticipants_;
            newQuest.transferOwnership(msg.sender);
            ++questIdCount;
            return address(newQuest);
        }

        revert QuestTypeInvalid();
    }
```

### [N-02] Insufficient coverage

**Description:**
The test coverage rate of the project is ~80%. Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.


```js

------------------------|----------|----------|----------|----------|----------------|
File                    |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------------------|----------|----------|----------|----------|----------------|
 contracts/             |    81.62 |    59.23 |    74.03 |    81.46 |                |
  Erc1155Quest.sol      |      100 |       75 |      100 |    85.71 |             35 |
  Erc20Quest.sol        |      100 |    66.67 |      100 |    94.12 |             60 |
  Quest.sol             |    90.91 |    81.58 |    73.33 |    90.48 |123,130,141,146 |
  QuestFactory.sol      |    94.29 |     72.5 |    85.71 |    94.23 |    146,173,200 |
  RabbitHoleReceipt.sol |    54.29 |    26.92 |    46.67 |    61.54 |... 184,185,193 |
  RabbitHoleTickets.sol |    58.33 |    18.75 |       50 |    59.09 |... 113,114,122 |
  ReceiptRenderer.sol   |      100 |      100 |      100 |      100 |                |
  TicketRenderer.sol    |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/  |      100 |      100 |      100 |      100 |                |
  IQuest.sol            |      100 |      100 |      100 |      100 |                |
  IQuestFactory.sol     |      100 |      100 |      100 |      100 |                |
 contracts/test/        |       25 |        0 |       50 |    28.57 |                |
  SampleERC20.sol       |      100 |      100 |      100 |      100 |                |
  SampleErc1155.sol     |      100 |      100 |      100 |      100 |                |
  TestERC20.sol         |        0 |        0 |        0 |        0 |  8,14,15,16,18 |
------------------------|----------|----------|----------|----------|----------------|
All files               |    78.47 |    57.46 |    72.84 |    79.72 |                |
------------------------|----------|----------|----------|----------|----------------|

```

### [N-03] NatSpec comments should be increased in contracts

**Context:**
All Contracts

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comments should be increased in contracts

### [N-04] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-05] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).
Consider adding a timelock to:

```solidity

contracts/QuestFactory.sol:
  185      /// @param questFee_ The quest fee value
  186:     function setQuestFee(uint256 questFee_) public onlyOwner {
  187:         if (questFee_ > 10_000) revert QuestFeeTooHigh();
  188:         questFee = questFee_;
  189:     }

```


### [N-06] Tokens accidentally sent to the contract cannot be recovered


It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```

### [N-07] Take advantage of Custom Error's return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can 
be written inside the `()` sign, this kind of approach provides a serious advantage in 
debugging and examining the revert details of dapps such as tenderly.


```solidity

24 results

contracts/Erc20Quest.sol:
  59          if (IERC20(rewardToken).balanceOf(address(this)) < maxTotalRewards() + maxProtocolReward())
  60:             revert TotalAmountExceedsBalance();
  61          super.start();

contracts/Erc1155Quest.sol:
  34          if (IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId) < totalParticipants)
  35:             revert TotalAmountExceedsBalance();
  36          super.start();

contracts/Quest.sol:
   34      ) {
   35:         if (endTime_ <= block.timestamp) revert EndTimeInPast();
   36:         if (startTime_ <= block.timestamp) revert StartTimeInPast();
   37:         if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
   38          endTime = endTime_;

   76      modifier onlyAdminWithdrawAfterEnd() {
   77:         if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
   78          _;

   82      modifier onlyStarted() {
   83:         if (!hasStarted) revert NotStarted();
   84          _;

   88      modifier onlyQuestActive() {
   89:         if (!hasStarted) revert NotStarted();
   90:         if (block.timestamp < startTime) revert ClaimWindowNotStarted();
   91          _;

   96      function claim() public virtual onlyQuestActive {
   97:         if (isPaused) revert QuestPaused();
   98  

  100  
  101:         if (tokens.length == 0) revert NoTokensToClaim();
  102  

  109  
  110:         if (redeemableTokenCount == 0) revert AlreadyClaimed();
  111  

  122      function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
  123:         revert MustImplementInChild();
  124      }

  129      function _transferRewards(uint256 amount_) internal virtual {
  130:         revert MustImplementInChild();
  131      }

contracts/QuestFactory.sol:
   69      ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
   70:         if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
   71  
   72          if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
   73:             if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
   74  

  105          if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
  106:             if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest();
  107  

  135  
  136:         revert QuestTypeInvalid();
  137      }

  165      function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
  166:         if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();
  167          protocolFeeRecipient = protocolFeeRecipient_;

  186      function setQuestFee(uint256 questFee_) public onlyOwner {
  187:         if (questFee_ > 10_000) revert QuestFeeTooHigh();
  188          questFee = questFee_;

  219      function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
  220:         if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
  221:         if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
  222:         if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
  223:         if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

```

### [N-08] Repeated validation logic can be converted into a function modifier

If a query or logic is repeated over many lines, using a modifier improves the readability and reusability of the code


```solidity

4 results 

contracts/QuestFactory.sol:
   72:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
  105:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {


```


### [N-09] For modern and more readable code; update import usages

**Context:**

```solidity

23 results - 5 files

contracts/QuestFactory.sol:
   3  
   4: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
   5  import {Erc20Quest} from './Erc20Quest.sol';

   9  import {OwnableUpgradeable} from '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
  10: import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
  11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
  12  

contracts/RabbitHoleReceipt.sol:
   3  
   4: import '@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol';
   5: import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721URIStorageUpgradeable.sol';
   6: import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';
   7: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
   8: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
   9: import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
  10: import '@openzeppelin/contracts-upgradeable/utils/CountersUpgradeable.sol';
  11: import './ReceiptRenderer.sol';
  12: import './interfaces/IQuestFactory.sol';
  13: import './interfaces/IQuest.sol';
  14  

contracts/RabbitHoleTickets.sol:
  3  
  4: import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
  5: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
  6: import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';
  7: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
  8: import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
  9: import './TicketRenderer.sol';
  10  

contracts/ReceiptRenderer.sol:
  3  
  4: import '@openzeppelin/contracts/utils/Base64.sol';
  5: import '@openzeppelin/contracts/utils/Strings.sol';
  6  

contracts/TicketRenderer.sol:
  3  
  4: import '@openzeppelin/contracts/utils/Base64.sol';
  5: import '@openzeppelin/contracts/utils/Strings.sol’;

```


**Description:**
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. 
This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation:**
`import {contract1 , contract2} from "filename.sol";`

A good example from the ArtGobblers project;
```js
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```
### [N-10] Use a more recent version of Solidity

**Context:**

```solidity

10 results 

contracts/Erc20Quest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/Erc1155Quest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/Quest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/QuestFactory.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/RabbitHoleReceipt.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/RabbitHoleTickets.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/ReceiptRenderer.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/TicketRenderer.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/interfaces/IQuest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/interfaces/IQuestFactory.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3 

```


**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md


**Recommendation:**
Old version of Solidity is used `(0.8.15)`, newer version can be used `(0.8.17)` 

### [N-11] For functions, follow Solidity standard naming conventions (internal function style rule)

**Context:**
```solidity

1 result 

contracts/QuestFactory.sol:
  152:     function grantDefaultAdminAndCreateQuestRole(address account_) internal {

```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)


### [N-12] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 
```solidity

10 results - 10 files

contracts/Erc20Quest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/Erc1155Quest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/Quest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/QuestFactory.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/RabbitHoleReceipt.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/RabbitHoleTickets.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/ReceiptRenderer.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/TicketRenderer.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.15;
  3  

contracts/interfaces/IQuest.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3  

contracts/interfaces/IQuestFactory.sol:
  1  // SPDX-License-Identifier: UNLICENSED
  2: pragma solidity ^0.8.15;
  3 

```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### [N-13] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

Project has upgrade scenario but hasn’t Stop scenario

### [N-14] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

### [N-15]  Open TODO


```solidity
contracts/interfaces/IQuest.sol:
  3  
  4: // TODO clean this whole thing up
```

**Recommendation:**
Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.


### [N-16] Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

There are many examples, some of which are listed below;

[RabbitHoleReceipt.sol#L192](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L192)

[TimeswapV2Option.sol#L85](https://github.com/code-423n4/2023-01-timeswap/blob/main/packages/v2-option/src/TimeswapV2Option.sol#L85)

### [N-17] No same value input control


```solidity

13 results

contracts/QuestFactory.sol:
  159:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
  165:     function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
  172:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
  179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

contracts/RabbitHoleReceipt.sol:
  65:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
  71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  77:     function setQuestFactory(address questFactory_) public onlyOwner {
  83:     function setMinterAddress(address minterAddress_) public onlyOwner {
  90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {

contracts/RabbitHoleTickets.sol:
  54:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {
  60:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  66:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  73:     function setMinterAddress(address minterAddress_) public onlyOwner {

```

### [N-18]  Add NatSpec Mapping comment

**Description:**
Add NatSpec comments describing mapping keys and values


**Context:**

```solidity

6 results - 3 files

contracts/Quest.sol:
  23  
  24:     mapping(uint256 => bool) private claimedList;
  25  

contracts/QuestFactory.sol:
  19      struct Quest {
  20:         mapping(address => bool) addressMinted;
  21          address questAddress;

  27      address public protocolFeeRecipient;
  28:     mapping(string => Quest) public quests;
  29      RabbitHoleReceipt public rabbitholeReceiptContract;
  30:     mapping(address => bool) public rewardAllowlist;
  31      uint public questFee;

contracts/RabbitHoleReceipt.sol:
  29      // storage
  30:     mapping(uint => string) public questIdForTokenId;
  31      address public royaltyRecipient;

  33      uint public royaltyFee;
  34:     mapping(uint => uint) public timestampForTokenId;
  35      ReceiptRenderer public ReceiptRendererContract;

```
**Recommendation:**

```solidity

 /// @dev    address(user) -> address(ERC0 Contract Address) -> uint256(allowance amount from user)
    mapping(address => mapping(address => uint256)) public allowance;

```


## [N-19] Include proxy contracts to Audit

The Proxy contract could not be seen in the checklist because it is an upgradable contract, only the implementation contracts are visible, I recommend including the Proxy contract in the audit for integrity in the audit


### [N-20] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```solidity

contracts/RabbitHoleReceipt.sol:
  82      /// @param minterAddress_ the address of the minter
  83:     function setMinterAddress(address minterAddress_) public onlyOwner {
  84:         minterAddress = minterAddress_;
  85:         emit MinterAddressSet(minterAddress_);
  86:     }
  87: 
  88:     /// @dev set the royalty fee
  89:     /// @param royaltyFee_ the royalty fee
  90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  91:         royaltyFee = royaltyFee_;
  92:         emit RoyaltyFeeSet(royaltyFee_);
  93:     }

```

### [N-21] Mark visibility of initialize(...) functions as ``external``

```solidity

contracts/QuestFactory.sol:
  36  
  37:     function initialize(
  38:         address claimSignerAddress_,
  39:         address rabbitholeReceiptContract_,
  40:         address protocolFeeRecipient_
  41:     ) public initializer {

contracts/RabbitHoleReceipt.sol:
  42  
  43:     function initialize(
  44:         address receiptRenderer_,
  45:         address royaltyRecipient_,
  46:         address minterAddress_,
  47:         uint royaltyFee_
  48:     ) public initializer {

contracts/RabbitHoleTickets.sol:
  31  
  32:     function initialize(
  33:         address ticketRenderer_,
  34:         address royaltyRecipient_,
  35:         address minterAddress_,
  36:         uint royaltyFee_
  37:     ) public initializer {


```

**Description:**
If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}_init function, not the parent public initialize(...) function.

External instead of public would give more the sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally)

Security point of view, it might be safer so that it cannot be called internally by accident in the child contract 

It might cost a bit less gas to use external over public 


It is possible to override a function from external to public (= "opening it up") ✅
but it is not possible to override a function from public to external (= "narrow it down"). ❌

For above reasons you can change initialize(...) to external


https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

### [N-22] Use of bytes.concat() instead of abi.encodePacked()

```solidity

7 results - 3 files

contracts/QuestFactory.sol:
  211:         bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));

contracts/ReceiptRenderer.sol:
   48:         bytes memory attributes = abi.encodePacked(
   63:         bytes memory dataURI = abi.encodePacked(
   83:         bytes memory attribute = abi.encodePacked(
  101:         bytes memory svg = abi.encodePacked(

contracts/TicketRenderer.sol:
  19:         bytes memory dataURI = abi.encodePacked(
  37:         bytes memory svg = abi.encodePacked(

```
Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

### [N-23] Make the Test Context with Solidity


```js

README.md:
  136: ├── test
  137: │   ├── Erc1155Quest.spec.ts
  138: │   ├── Erc20Quest.spec.ts
  139: │   ├── Quest.spec.ts
  140: │   ├── QuestFactory.spec.ts
  141: │   ├── RabbitHoleReceipt.spec.ts
  142: │   ├── RabbitHoleTickets.spec.ts
  143: │   ├── SampleErc1155.spec.ts
  144: │   ├── SampleErc20.spec.ts
  145: │   └── types.ts
  146  ├── test-gas-stories

```

It's crucial to write tests with possibly 100% coverage for smart contract systems.

It is recommended to write appropriate tests for all possible code streams and especially for extreme cases.

But the other important point is the test context, tests written with solidity are safer, it is recommended to focus on tests with Foundry

### [N-24] Keccak Constant values should used to immutable rather than constant

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand.



```solidity
1 result

contracts/QuestFactory.sol:
  17:     bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE’);

```
