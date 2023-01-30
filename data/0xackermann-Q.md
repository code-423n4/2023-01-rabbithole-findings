### QA Report

### Low Risk Issues
| No. |  Issue |
| --- | --- | 
| 1 | [Missing calls to `__ReentrancyGuard_init` functions of inherited contracts](#L-01-Missing-calls-to-__ReentrancyGuard_init-functions-of-inherited-contracts) |
| 2 | [Royalty Fee upper limit](#L-02-Royalty-Fee-upper-limit) |
| 3 | [Owner can renounce Ownership](#L-03-Owner-can-renounce-Ownership) |
| 4 | [Loss of precision due to rounding](#L-04-Loss-of-precision-due-to-rounding) |
| 5 | [Use `safeTransferOwnership` instead of `transferOwnership` function](#L-05-Use-safeTransferOwnership-instead-of-transferOwnership-function) |

### Non Critical Risk Issues
| No. | Issue |
| --- | --- |
| 1 | [For modern and more readable code; update import usages](#NC-01-for-modern-and-more-readable-code-update-import-usages)|
| 2 | [Include return parameters in NatSpec comments](#NC-02-Include-return-parameters-in-NatSpec-comments) |
| 3 | [Return parameters are declared but not used](#NC-03-Return-parameters-are-declared-but-not-used) |
| 4 | [Parameter is not used in the function](#NC-04-Parameter-is-not-used-in-the-function) | 
| 5 | [Use a more recent version of Solidity](#NC-05-Use-a-more-recent-version-of-Solidity) |
| 6 | [Constant values such as a call to `keccak256()`, should used to immutable rather than constant](#NC-06-Constant-values-such-as-a-call-to-keccak256-should-used-to-immutable-rather-than-constant) |
| 7| [Floating pragma](#NC-07-Floating-pragma) |
| 8 | [Open TODOs](#NC-08-Open-TODOs) |
| 9 | [Long lines are not suitable for the ‘Solidity Style Guide’](#NC-09-Long-lines-are-not-suitable-for-the-Solidity-Style-Guide) |
| 10 | [Signature scheme does not support smart contracts](#NC-10-Signature-scheme-does-not-support-smart-contracts) | 
| 11 | [Omissions in Events](#NC-11-Omissions-in-Events) |


---
### [L-01] Missing calls to `__ReentrancyGuard_init` functions of inherited contracts

---

**Description:**

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of initialize() functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.

**Lines of Code:**
- [QuestFactory.sol#L37-L50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37-L50)

Recommended Mitigation Steps
Add missing calls to init functions of inherited contracts.

```diff
 ) public initializer {
    __Ownable_init();
    __AccessControl_init();
+   __ReentrancyGuard_init();
    grantDefaultAdminAndCreateQuestRole(msg.sender);
    claimSignerAddress = claimSignerAddress_;
    rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
    setProtocolFeeRecipient(protocolFeeRecipient_);
    setQuestFee(2_000);
    questIdCount = 1;
}
```

---
### [L-02] Royalty Fee upper limit

---
**Description:**

The idea is the same for `questFee` which has a upper limit of 10000, which can be found at [QuestFactory.sol#L187](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187). This is to avoid to have high royalty fees. 

```solidity
function setQuestFee(uint256 questFee_) public onlyOwner {
    if (questFee_ > 10_000) revert QuestFeeTooHigh();
    questFee = questFee_;
}
```

**Recommendation:**
```solidity
function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
    if (royaltyFee_ > 10_000) revert RoyaltyFeeTooHigh(); 
    royaltyFee = royaltyFee_;
    emit RoyaltyFeeSet(royaltyFee_);
}
```

**Lines of Code:** 
- [RabbitHoleReceipt.sol#L90-L93](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90-L93)
- [RabbitHoleTickets.sol#L66-L69](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66-L69)


--- 
### [L-03] Owner can renounce Ownership
---


**Description:** 

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The non-fungible Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

**Lines of Code:**
- [QuestFactory.sol#L9](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9)
- [RabbitHoleReceipt.sol#L7](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L7)
- [RabbitHoleTickets.sol#L5](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L5)


`onlyOwner` functions:
- [Erc1155Quest.sol#L54](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54)
- [Erc20Quest.sol#L81](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81)
- [Quest.sol#L50](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)
- [Quest.sol#L57](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57)
- [Quest.sol#L63](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L63)
- [Quest.sol#L150](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150)
- [QuestFactory.sol#L9](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9)
- [QuestFactory.sol#L142](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142)
- [QuestFactory.sol#L159](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159)
- [QuestFactory.sol#L165](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165)
- [QuestFactory.sol#L172](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172)
- [QuestFactory.sol#L179](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179)
- [QuestFactory.sol#L186](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186)
- [RabbitHoleReceipt.sol#L7](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L7)
- [RabbitHoleReceipt.sol#L65](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65)
- [RabbitHoleReceipt.sol#L71](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71)
- [RabbitHoleReceipt.sol#L77](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77)
- [RabbitHoleReceipt.sol#L83](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83)
- [RabbitHoleReceipt.sol#L90](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90)
- [RabbitHoleTickets.sol#L5](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L5)
- [RabbitHoleTickets.sol#L54](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54)
- [RabbitHoleTickets.sol#L60](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60)
- [RabbitHoleTickets.sol#L66](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66)
- [RabbitHoleTickets.sol#L73](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73)

--- 
### [L-04] Loss of precision due to rounding

---

```solidity
return (maxTotalRewards() * questFee) / 10_000;
```

**Lines of Code:**

- [Erc20Quest.sol#L53](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L53)
- [Erc20Quest.sol#L97](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L97)
- [RabbitHoleReceipt.sol#L184](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184)
- [RabbitHoleTickets.sol#L113](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113)

---
### [L-05] Use `safeTransferOwnership` instead of `transferOwnership` function

---

**Description:**
```transferOwnership``` function is used to change Ownership from ```Ownable.sol```.

Use a 2 structure transferOwnership which is safer. 
```safeTransferOwnership```,  use it is more secure due to 2-stage ownership transfer.

**Recommendation:**
Use ``Ownable2Step.sol``
[Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)

**Lines of Code:**
- [QuestFactory.sol#L100](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L100)
- [QuestFactory.sol#L131](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L131)


---
### [NC-01] For modern and more readable code; update import usages

---
**Description:**

Our Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

**Recommendation:** 
```solidity
import {contract1 , contract2} from "filename.sol";
```

**Lines of Code:** 

- [QuestFactory.sol#L4-L11](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L4-L11)
- [RabbitHoleReceipt.sol#L4-L13](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L4-L13)
- [RabbitHoleTickets.sol#L4-L9](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L4-L9)
- [ReceiptRenderer.sol#L4-L5](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L4-L5)
- [TicketRenderer.sol#L4-L5](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L4-L5)

---
### [NC-02] Include return parameters in NatSpec comments

---

**Description:**

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

If Return parameters are declared, you must prefix them with "/// @return".

Some code analysis programs do analysis by reading NatSpec details, if they can't see the "@return" tag, they do incomplete analysis.

**Recommendation:** 
Include return parameters in NatSpec comments

*Recommendation Code Style: (from Uniswap3)*
```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```

**Lines of Code:**
- [RabbitHoleReceipt.sol#L175-L186](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L175-L186)
- [RabbitHoleTickets.sol#L106-L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L106-L115)


---
### [NC-03] Return parameters are declared but not used

---

**Recommendation:** 
```diff
  address receiver, uint256 royaltyAmount) {
    require(_exists(tokenId_), 'Nonexistent token');

-    uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
-    return (royaltyRecipient, royaltyPayment);

+    receiver = royaltyRecipient;
+    royaltyAmount = (salePrice_ * royaltyFee) / 10_000;
  }
```

**Lines of Code:**
- [RabbitHoleReceipt.sol#L175-L186](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L175-L186)
- [RabbitHoleTickets.sol#L106-L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L106-L115)


---
### [NC-04] Parameter is not used in the function

---

`tokenId_` from [RabbitHoleTickets.sol#L110](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L110) is not used in the function.

```solidity
function royaltyInfo(
    uint256 tokenId_,
    uint256 salePrice_
) external view override returns (address receiver, uint256 royaltyAmount) {
    uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
    return (royaltyRecipient, royaltyPayment);
}
```

---
### [NC-05] Use a more recent version of Solidity

---


**Description:** 

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

**Recommendation:** 

Old version of Solidity is used , newer version can be used `(0.8.17)`

**Lines of Code:** All contracts

---
### [NC-06] Constant values such as a call to `keccak256()`, should used to immutable rather than constant

---

**Description:**

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

**Lines of Code:**
- [QuestFactory.sol#L17](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17)

```solidity
bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```

---
### [NC-07] Floating pragma

---

**Description:**

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

https://swcregistry.io/docs/SWC-103

**Recommendation:**

Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

**Floating Pragma List:**
`pragma solidity ^0.8.15;` (all contracts)

---
### [NC-08] Open TODOs

---

**Recommendation:**
Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.

**Lines of code:**
- [IQuest.sol#L4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4)

---
### [NC-09] Long lines are not suitable for the ‘Solidity Style Guide’

---

**Description:**

It is generally recommended that lines in the source code should not exceed 80-120 characters. Today's screens are much larger, so in some cases it makes sense to expand that. The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.

(why-is-80-characters-the-standard-limit-for-code-width)[https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width]

**Recommendation:** 

Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction

**Lines of code:**
- [RabbitHoleReceipt.sol#L172](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L172)
- [ReceiptRenderer.sol#L108](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L108)
- [IQuestFactory.sol#L16](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L16)

---
### [NC-10] Signature scheme does not support smart contracts

---

**Description:**

Non-Fungible does not support EIP 1271 and therefore no signatures that are validated by smart contracts. This limits the applicability for protocols that want to build on top of it and persons that use smart contract wallets. Consider implementing support for it https://eips.ethereum.org/EIPS/eip-1271


---
### [NC-11] Omissions in Events

---

**Descriptions:**

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

**Recommendation:**

The events should include the new value and old value where possible:
Events with no old value;;

**Lines of Code:**
- [RabbitHoleReceipt.sol#L85](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L85)
- [RabbitHoleReceipt.sol#L92](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L92)
- [RabbitHoleTickets.sol#L68](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L68)
- [RabbitHoleTickets.sol#L75](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L75)
