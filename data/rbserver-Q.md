## [01] `questIdCount` CAN BE MISLEADING
As shown by the following `QuestFactory.initialize` function, `questIdCount` is set to 1 when the `QuestFactory` contract is initialized; however, no quests are created at that moment. Later, when the `createQuest` function below is called, `questIdCount` is incremented by 1. This means that `questIdCount` is always 1 larger than the number of created quests. Since `questIdCount` is public, users can all view it but it can provide misleading information about the number of quests that have been created. To avoid confusion, please consider not setting `questIdCount` to 1 in the `QuestFactory.initialize` function.

https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/QuestFactory.sol#L32
```solidity
    uint public questIdCount;
```

https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/QuestFactory.sol#L37-L50
```solidity
    function initialize(
        address claimSignerAddress_,
        address rabbitholeReceiptContract_,
        address protocolFeeRecipient_
    ) public initializer {
        ...
        questIdCount = 1;
    }
```

https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/QuestFactory.sol#L61-L137
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
        ...

        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
            ...
            ++questIdCount;
            ...
        }

        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
            ...
            ++questIdCount;
            ...
        }

        ...
    }
```

## [02] `_setClaimed` FUNCTION CAN BE CALLED FOR TOKENS THAT ARE ALREADY MARKED AS CLAIMED
When calling the following `claim` function, `tokens` that are used as the input for the `_setClaimed` function call includes `msg.sender`'s all RabbitHole receipt tokens for the corresponding quest. Some of these tokens are already marked as claimed during the previous claims so executing `_setClaimed(tokens)` can mark the tokens, which have the associated rewards already claimed, as claimed again. To avoid such redundant and unnecessary operations, please consider updating the `_setClaimed` function to skip the tokens that are already marked as claimed.

https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L96-L118
```solidity
    function claim() public virtual onlyQuestActive {
        ...

        uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

        if (tokens.length == 0) revert NoTokensToClaim();

        uint256 redeemableTokenCount = 0;
        for (uint i = 0; i < tokens.length; i++) {
            if (!isClaimed(tokens[i])) {
                redeemableTokenCount++;
            }
        }

        if (redeemableTokenCount == 0) revert AlreadyClaimed();

        uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
        _setClaimed(tokens);
        ...
    }
```

https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L69-L73
```solidity
    function _setClaimed(uint256[] memory tokenIds_) private {
        for (uint i = 0; i < tokenIds_.length; i++) {
            claimedList[tokenIds_[i]] = true;
        }
    }
```

## [03] MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS
( Please note that the following instances are not found in https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480#nc-1-missing-checks-for-address0-when-assigning-values-to-address-state-variables. )

To prevent unintended behaviors, critical address inputs should be checked against `address(0)`.

`address(0)` check is missing for `to_` in the following `withdrawRemainingTokens` function. Please consider checking it.
https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Erc20Quest.sol#L81-L87
```solidity
    function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_);

        ...
        IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);
    }
```

`address(0)` check is missing for `rabbitholeReceiptContract_` in the following `initialize` function. Please consider checking it.
https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/QuestFactory.sol#L37-L50
```solidity
    function initialize(
        address claimSignerAddress_,
        address rabbitholeReceiptContract_,
        address protocolFeeRecipient_
    ) public initializer {
        ...
        rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
        ...
    }
```

`address(0)` check is missing for `rabbitholeReceiptContract_` in the following `setRabbitHoleReceiptContract` function. Please consider checking it.
https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/QuestFactory.sol#L172-L174
```solidity
    function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
        rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
    }
```

`address(0)` check is missing for `minterAddress_` in the following `initialize` function. Please consider checking it.
https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L32-L45
```solidity
    function initialize(
        address ticketRenderer_,
        address royaltyRecipient_,
        address minterAddress_,
        uint royaltyFee_
    ) public initializer {
        ...
        minterAddress = minterAddress_;
        ...
    }
```

`address(0)` check is missing for `minterAddress_` in the following `setMinterAddress` function. Please consider checking it.
https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L73-L76
```solidity
    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```

## [04] CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS
To improve readability and maintainability, a constant can be used instead of the magic number. Please consider replacing the magic numbers, such as `10_000`, used in the following code with constants.

```solidity
quest-protocol\contracts\Erc20Quest.sol
  53: return (maxTotalRewards() * questFee) / 10_000;
  97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;

quest-protocol\contracts\QuestFactory.sol
  48: setQuestFee(2_000);
  187: if (questFee_ > 10_000) revert QuestFeeTooHigh();

quest-protocol\contracts\RabbitHoleReceipt.sol
  184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

quest-protocol\contracts\RabbitHoleTickets.sol
  113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

## [05] REDUNDANT NAMED RETURNS
When a function has unused named returns and used return statement, these named returns become redundant. To improve readability and maintainability, these variables for the named returns can be removed while keeping the return statements for the following `royaltyInfo` functions.

https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleReceipt.sol#L178-L186
```solidity
    function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount) {
        ...
        return (royaltyRecipient, royaltyPayment);
    }
```

https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L109-L115
```solidity
    function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount) {
        ...
        return (royaltyRecipient, royaltyPayment);
    }
```

## [06] `uint256` CAN BE USED INSTEAD OF `uint`
Both `uint` and `uint256` are used throughout the protocol's code. In favor of explicitness, please consider using `uint256` instead of `uint` in the following code.
```solidity
quest-protocol\contracts\Erc20Quest.sol
  84: uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;

quest-protocol\contracts\Quest.sol
  70: for (uint i = 0; i < tokenIds_.length; i++) {
  99: uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
  104: for (uint i = 0; i < tokens.length; i++) {

quest-protocol\contracts\QuestFactory.sol
  22: uint totalParticipants;
  23: uint numberMinted;
  31: uint public questFee;
  32: uint public questIdCount;
  193: function getNumberMinted(string memory questId_) external view returns (uint) {
  199: function questInfo(string memory questId_) external view returns (address, uint, uint) {
  199: function questInfo(string memory questId_) external view returns (address, uint, uint) {

quest-protocol\contracts\RabbitHoleReceipt.sol
  30: mapping(uint => string) public questIdForTokenId;
  33: uint public royaltyFee;
  34: mapping(uint => uint) public timestampForTokenId;
  34: mapping(uint => uint) public timestampForTokenId;
  47: uint royaltyFee_
  100: uint newTokenID = _tokenIds.current();
  112: ) public view returns (uint[] memory) {
  113: uint msgSenderBalance = balanceOf(claimingAddress_);
  114: uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
  114: uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
  115: uint foundTokens = 0;
  117: for (uint i = 0; i < msgSenderBalance; i++) {
  118: uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
  125: uint[] memory filteredTokens = new uint[](foundTokens);
  125: uint[] memory filteredTokens = new uint[](foundTokens);
  126: uint filterTokensIndexTracker = 0;
  128: for (uint i = 0; i < msgSenderBalance; i++) {
  159: uint tokenId_
  165: (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
  169: uint rewardAmount = questContract.getRewardAmount();

quest-protocol\contracts\RabbitHoleTickets.sol
  24: uint public royaltyFee;
  36: uint royaltyFee_
  102: function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {

quest-protocol\contracts\ReceiptRenderer.sol
  22: uint tokenId_,
  24: uint totalParticipants_,
  26: uint rewardAmount_,
  41: uint tokenId_,
  43: uint totalParticipants_,
  45: uint rewardAmount_,
  100: function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {

quest-protocol\contracts\TicketRenderer.sol
  17: uint tokenId_
  36: function generateSVG(uint tokenId_) public pure returns (string memory) {
```

## [07] INCOMPLETE NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss the `@param` and/or `@return` comments. Please consider completing the NatSpec comments for these functions.

```solidity
quest-protocol\contracts\Erc20Quest.sol
  96: function protocolFee() public view returns (uint256) {  

quest-protocol\contracts\Erc1155Quest.sol
  33: function start() public override {  

quest-protocol\contracts\Quest.sol
  135: function isClaimed(uint256 tokenId_) public view returns (bool) {   
  140: function getRewardAmount() public view returns (uint256) { 
  145: function getRewardToken() public view returns (address) { 
  150: function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {} 

quest-protocol\contracts\QuestFactory.sol
  193: function getNumberMinted(string memory questId_) external view returns (uint) { 
  199: function questInfo(string memory questId_) external view returns (address, uint, uint) { 
  210: function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) { 

quest-protocol\contracts\RabbitHoleReceipt.sol
  109: function getOwnedTokenIdsOfQuest(   
  190: function supportsInterface( 

quest-protocol\contracts\RabbitHoleTickets.sol
  102: function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {  
  119: function supportsInterface( 
```

## [08] MISSING NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss NatSpec comments. Please consider adding NatSpec comments for these functions.

```solidity
quest-protocol\contracts\QuestFactory.sol
  37: function initialize(    

quest-protocol\contracts\RabbitHoleReceipt.sol
  43: function initialize(    
  158: function tokenURI(  

quest-protocol\contracts\ReceiptRenderer.sol
  40: function generateDataURI(   
```