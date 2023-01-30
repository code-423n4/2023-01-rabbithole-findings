
### 1. FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES
### Description

Our Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

### Recommendation

`import {contract1 , contract2} from "filename.sol";`

POC:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
4: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
10: import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
4-13: import '@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol';
import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721URIStorageUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
import '@openzeppelin/contracts-upgradeable/utils/CountersUpgradeable.sol';
import './ReceiptRenderer.sol';
import './interfaces/IQuestFactory.sol';
import './interfaces/IQuest.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
4-9: import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
import './TicketRenderer.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol

```
4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';
```


### 2. CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
48: setQuestFee(2_000);
187: if (questFee_ > 10_000) revert QuestFeeTooHigh();
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

### 3. EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constants` should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the constructor.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
17: bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```


### 4. LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
187: if (questFee_ > 10_000) revert QuestFeeTooHigh();
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```


### 5. INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

*Number of Instances Identified: 17*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
96: function protocolFee() public view returns (uint256)
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
122: function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256)
135: function isClaimed(uint256 tokenId_) public view returns (bool)
140: function getRewardAmount() public view returns (uint256)
145: function getRewardToken() public view returns (address)
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
193: function getNumberMinted(string memory questId_) external view returns (uint)
199: function questInfo(string memory questId_) external view returns (address, uint, uint)
210: function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address)
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
109: function getOwnedTokenIdsOfQuest()
158: function tokenURI()
181: function royaltyInfo()
192: function supportsInterface()
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
102: function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory)
112: function royaltyInfo()
119: function supportsInterface()
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
40: function generateDataURI()
82: function generateAttribute(string memory key, string memory value) public pure returns (string memory)
```

### Description

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

[https://docs.soliditylang.org/en/v0.8.15/natspec-format.html](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

If Return parameters are declared, you must prefix them with `/// @return`.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the `@return` tag, they do incomplete analysis.

### Recommendation

Include return parameters in NatSpec comments

Recommendation Natspec Code Style:

```
   /// @return Returns a page's URI if it has been minted
```


### 6. MISSING EVENTS FOR FUNCTIONS THAT CHANGE CRITICAL PARAMETERS

*Number of Instances Identified: 16*

The functions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

Missing events and timelocks do not promote transparency and if such changes immediately affect users’ perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
81: function withdrawRemainingTokens(address to_) public override onlyOwner
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
50: function start() public virtual onlyOwner
57: function pause() public onlyOwner onlyStarted
63: function unPause() public onlyOwner onlyStarted
150: function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
142: function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner
159: function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner
165: function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner
172: function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner
179: function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
65: function setReceiptRenderer(address receiptRenderer_) public onlyOwner
71: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner
77: function setQuestFactory(address questFactory_) public onlyOwner
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
54: function setTicketRenderer(address ticketRenderer_) public onlyOwner
60: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner
83: function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter
```


### 7. UPGRADEABLE CONTRACT IS MISSING A `__GAP[50]` STORAGE VARIABLE TO ALLOW FOR NEW STORAGE VARIABLES IN LATER VERSIONS

*Number of Instances Identified: 3*

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

POC:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
16: contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
15-21: contract RabbitHoleReceipt is
    Initializable,
    ERC721Upgradeable,
    ERC721EnumerableUpgradeable,
    ERC721URIStorageUpgradeable,
    OwnableUpgradeable,
    IERC2981Upgradeable
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
11-16: contract RabbitHoleTickets is
    Initializable,
    ERC1155Upgradeable,
    OwnableUpgradeable,
    ERC1155BurnableUpgradeable,
    IERC2981Upgradeable
```


### 8. OPEN TODOS

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

```
4: // TODO clean this whole thing up
```


### 9. USE LATEST SOLIDITY VERSION

When deploying contracts, you should use the latest released version of Solidity. Apart from exceptional cases, only the latest version receives [security fixes](https://github.com/ethereum/solidity/security/policy#supported-versions). Furthermore, breaking changes as well as new features are introduced regularly.
Latest Version is 0.8.17

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol

```
pragma solidity ^0.8.15;
```


### 10. FLOATING PRAGMA VERSION SHOULD NOT BE USED

n the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

```
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol

```
pragma solidity ^0.8.15;
```


### 11. ADDING A RETURN STATEMENT WHEN THE FUNCTION DEFINES A NAMED RETURN VARIABLE, IS REDUNDANT

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
178-181: function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount)
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
109-112: function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount)
```

