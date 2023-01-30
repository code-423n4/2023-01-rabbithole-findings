# [QA] - Use enums for contract type tracking instead of strings

In `QuestFactory.sol`, the way that the contract decides which version of the quest will deploy is inefficient since it's using a string param `contractType_`. Because the amount of options that we have are limited it would be better to use an enum instead.

```diff
+    enum ContractTypes {
+        ERC20,
+        ERC1155
+    }

    contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {
        ...

        function createQuest(
            address rewardTokenAddress_,
            uint256 endTime_,
            uint256 startTime_,
            uint256 totalParticipants_,
            uint256 rewardAmountOrTokenId_,
-            string memory contractType_,
+           ContractTypes contractType_,
            string memory questId_
        ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {

-           if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
+           if (contractType_ == ContractTypes.ERC20)
                ...
            }

-           if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
+           if (contractType_ == ContractTypes.ERC1155)  
                ...
            }
        }
        ...
    }
```

# [QA] - Use [Clones.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/Clones.sol) for Quests deployment to save gas.

The current implementation on `ContractFactory` will deploy a completly new contract for every quest. This is not gas efficient since the implementation used for each quest type is always the same (though, could be upgraded for future quests). A better approach will be to use Openzeppelin's Clones.sol where a single implementation contract is deployed and a lighter transparent proxy is deployed later for each new quest pointing to that single implementation contract.

Source: https://blog.openzeppelin.com/workshop-recap-cheap-contract-deployment-through-clones/

An implementation example on (QuestFactory.sol)[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol]:

```diff
    contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {



+       mapping(ContractTypes => address) private implementations;        

+       event ImplementationUpdated(
+            ContractTypes indexed contractType,
+            address implementation,
+            uint256 timestamp
+       );


+        constructor() {
+            _initializeImplementations();
+            _disableInitializers();
+        }

+       function _initializeImplementations() private {
+          setContractImplementation(
+               ContractTypes.ERC20,
+               address(new ERC20Quest())
+           );
+           setContractImplementation(
+               ContractTypes.ERC1155,
+               address(new ERC1155Quest())
+           );
+       }

+        function setContractImplementation(
+            ContractTypes contractType,
+            address implementation
+        ) public onlyOwner {
+            implementations[contractType] = implementation;
+            emit ImplementationUpdated(
+                contractType,
+                implementation,
+                block.timestamp
+            );
+        }

        function createQuest(
            address rewardTokenAddress_,
            uint256 endTime_,
            uint256 startTime_,
            uint256 totalParticipants_,
            uint256 rewardAmountOrTokenId_,
-            string memory contractType_,
+           ContractTypes contractType_,
            string memory questId_
        ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
+           address implementation = implementations[contractType];

+           if (implementation == address(0)) {
+             revert InvalidImplementation();
+           }

+           address newQuestAddress = Clones.clone(implementation);
+           Quest(newQuestAddress).initialize(
              ...
+           );

            ...

            return newQuestAddress;
        }
    }
```

* It's required to convert the quest contracts to initializable ones (moving the logic from the constructors to the initialize function.)

# [QA] - `QuestFactory.sol` constructor should include a `_initializeImplementations()` call.

It's recommended to call the function `_initializeImplementations()` in the constructor on initalizable contracts.
This would prevent that the reference implementation contract would be initialized for unauthorized third parties.

```diff
      /// @custom:oz-upgrades-unsafe-allow constructor
-   constructor() initializer {}
+   constructor() {
+       _initializeImplementations();
+   }
```

# [QA] - Use inheritance over composition on `RabbitHoleReceipt.sol` and `RabbitHoleTickets.sol`.

Currently an auxiliar contract is being deployed for the NFT contracts with renderer functionality. This will add a gas overhead (since it would require to deploy two completly separated contracts and will add external calls to them) and adds unnecesary complexity to those contracts.
A simpler approach would be to use inheritance instead.

Example:

`RabitHoleReceipt.sol`

```diff

-   import './ReceiptRenderer.sol';
+   import {ReceiptRenderer} from './ReceiptRenderer.sol';

    contract RabbitHoleReceipt is
      Initializable,
      ERC721Upgradeable,
      ERC721EnumerableUpgradeable,
      ERC721URIStorageUpgradeable,
      OwnableUpgradeable,
      IERC2981Upgradeable,
+     ReceiptRenderer
    {

-       ReceiptRenderer public ReceiptRendererContract;     

        function initialize(
-           address receiptRenderer_,
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
-           ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
        }

        /// @dev set the receipt renderer contract
        /// @param receiptRenderer_ the address of the receipt renderer contract
-       function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
-           ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
-       }

        /// @dev return the token uri, this delegates to the receipt renderer contract
        function tokenURI(
            uint tokenId_
        ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {
            require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');
            require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');

            string memory questId = questIdForTokenId[tokenId_];
            (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
            IQuest questContract = IQuest(questAddress);

            bool claimed = questContract.isClaimed(tokenId_);
            uint rewardAmount = questContract.getRewardAmount();
            address rewardAddress = questContract.getRewardToken();

-           return ReceiptRendererContract.generateTokenURI(tokenId_, questId, totalParticipants, claimed, rewardAmount, rewardAddress);
+           return generateTokenURI(tokenId_, questId, totalParticipants, claimed, rewardAmount, rewardAddress);
        }
    }
```
