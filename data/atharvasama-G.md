# Gas Optimizations list

| Number | Details                                                                             | Instances |
| ------ | ----------------------------------------------------------------------------------- | --------- |
| 1      | ```x += y``` costs more gas than ```x = x + y``` for state variables                | 1         |
| 2      | Can declare the variable outside the loop to save gas                               | 4         |
| 3      | Public functions not called by contract can be changed to external to save some gas | 31        |
| 4      | Don't compare boolean expressions to boolean literals                               | 3         |
| 5      | Internal functions only called once can be inlined to save gas                      | 1         |
| 6      | Multiple address/id mappings can be combined into a single mapping to a struct      | 2         |




# Gas Optimizations
## [G-01] ```x += y``` costs more gas than ```x = x + y``` for state variables.
Using the addition operator instead of plus-equals saves some gas.
>quest-protocol\contracts\Quest.sol:
>```js
>114          _transferRewards(totalRedeemableRewards);
>115:         redeemedTokens += redeemableTokenCount;
>116  
>```

## [G-02] Can declare the variable outside the loop to save gas
Declaring the loop variable outside the loop saves gas.

>quest-protocol\contracts\Quest.sol:
>```js
>69      function _setClaimed(uint256[] memory tokenIds_) private {
>70:         for (uint i = 0; i < tokenIds_.length; i++) {
>71              claimedList[tokenIds_[i]] = true;
>
>103          uint256 redeemableTokenCount = 0;
>104:         for (uint i = 0; i < tokens.length; i++) {
>105              if (!isClaimed(tokens[i])) {
>```
>quest-protocol\contracts\RabbitHoleReceipt.sol:
>```js
>116  
>117:         for (uint i = 0; i < msgSenderBalance; i++) {
>118              uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
>
>127  
>128:         for (uint i = 0; i < msgSenderBalance; i++) {
>129              if (tokenIdsForQuest[i] > 0) {
>```

## [G-03] Public functions not called by contract can be changed to external to save some gas
Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so.

>quest-protocol\contracts\Erc20Quest.sol
>```js
>102:     function withdrawFee() public onlyAdminWithdrawAfterEnd {
>103:         IERC20(rewardToken).safeTransfer(protocolFeeRecipient, protocolFee());
>104:     }
>```
>
>quest-protocol\contracts\Quest.sol
>```js
>50:     function start() public virtual onlyOwner {
>51:         isPaused = false;
>52:         hasStarted = true;
>53:     }
>
>57:     function pause() public onlyOwner onlyStarted {
>58:         isPaused = true;
>59:     }
>
>63:     function unPause() public onlyOwner onlyStarted {
>64:         isPaused = false;
>65:     }
>
>96:     function claim() public virtual onlyQuestActive {
>
>140:     function getRewardAmount() public view returns (uint256) {
>141:         return rewardAmountInWeiOrTokenId;
>142:     }
>
>145:     function getRewardToken() public view returns (address) {
>146:         return rewardToken;
>147:     }
>```
>
>File: quest-protocol\contracts\QuestFactory.sol
>```js
>37:     function initialize(
>38:         address claimSignerAddress_,
>39:         address rabbitholeReceiptContract_,
>40:         address protocolFeeRecipient_
>41:     ) public initializer {
>
>61:     function createQuest(
>62:         address rewardTokenAddress_,
>63:         uint256 endTime_,
>64:         uint256 startTime_,
>65:         uint256 totalParticipants_,
>66:         uint256 rewardAmountOrTokenId_,
>67:         string memory contractType_,
>68:         string memory questId_
>69:     ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
>
>142:     function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
>
>159:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
>160:         claimSignerAddress = claimSignerAddress_;
>161:     }
>
>172:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
>173:         rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
>174:     }
>
>179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
>180:         rewardAllowlist[rewardAddress_] = allowed_;
>181:     }
>
>219:     function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
>```
>
>quest-protocol\contracts\RabbitHoleReceipt.sol
>```js
>43:     function initialize(
>44:         address receiptRenderer_,
>45:         address royaltyRecipient_,
>46:         address minterAddress_,
>47:         uint royaltyFee_
>48:     ) public initializer {
>
>71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
>72:         royaltyRecipient = royaltyRecipient_;
>73:     }
>
>77:     function setQuestFactory(address questFactory_) public onlyOwner {
>78:         QuestFactoryContract = IQuestFactory(questFactory_);
>79:     }
>
>83:     function setMinterAddress(address minterAddress_) public onlyOwner {
>84:         minterAddress = minterAddress_;
>85:         emit MinterAddressSet(minterAddress_);
>86:     }
>
>90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
>91:         royaltyFee = royaltyFee_;
>92:         emit RoyaltyFeeSet(royaltyFee_);
>93:     }
>
>098:     function mint(address to_, string memory questId_) public onlyMinter {
>099:         _tokenIds.increment();
>100:         uint newTokenID = _tokenIds.current();
>101:         questIdForTokenId[newTokenID] = questId_;
>102:         timestampForTokenId[newTokenID] = block.timestamp;
>103:         _safeMint(to_, newTokenID);
>104:     }
>
>109:     function getOwnedTokenIdsOfQuest(
>110:         string memory questId_,
>111:         address claimingAddress_
>112:     ) public view returns (uint[] memory) {
>
>158:     function tokenURI(
>159:         uint tokenId_
>160:     ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {
>```
>
>quest-protocol\contracts\RabbitHoleTickets.sol
>```js
>32:     function initialize(
>33:         address ticketRenderer_,
>34:         address royaltyRecipient_,
>35:         address minterAddress_,
>36:         uint royaltyFee_
>37:     ) public initializer {
>
>54:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {
>55:         TicketRendererContract = TicketRenderer(ticketRenderer_);
>56:     }
>
>60:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
>61:         royaltyRecipient = royaltyRecipient_;
>62:     }
>
>66:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
>67:         royaltyFee = royaltyFee_;
>68:         emit RoyaltyFeeSet(royaltyFee_);
>69:     }
>
>73:     function setMinterAddress(address minterAddress_) public onlyOwner {
>74:         minterAddress = minterAddress_;
>75:         emit MinterAddressSet(minterAddress_);
>76:     }
>
>83:     function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
>84:         _mint(to_, id_, amount_, data_);
>85:     }
>
>92:     function mintBatch(
>93:         address to_,
>94:         uint256[] memory ids_,
>95:         uint256[] memory amounts_,
>96:         bytes memory data_
>97:     ) public onlyMinter {
>98:         _mintBatch(to_, ids_, amounts_, data_);
>99:     }
>
>102:     function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
>103:         return TicketRendererContract.generateTokenURI(tokenId_);
>104:     }
>```
>
>quest-protocol\contracts\ReceiptRenderer.sol
>```js
>21:     function generateTokenURI(
>22:         uint tokenId_,
>23:         string memory questId_,
>24:         uint totalParticipants_,
>25:         bool claimed_,
>26:         uint rewardAmount_,
>27:         address rewardAddress_
>28:     ) public view virtual returns (string memory) {
>```
>
>quest-protocol\contracts\TicketRenderer.sol
>```js
>16:     function generateTokenURI(
>17:         uint tokenId_
>18:     ) public pure returns (string memory) {
>```

## [G-04] Don't compare boolean expressions to boolean literals
It is not required to check for equality between boolean expressions and boolean literals and can be ommitted to save gas. 
>quest-protocol\contracts\Quest.sol:
>```js
>136:         return claimedList[tokenId_] == true;
>```
>
>quest-protocol\contracts\QuestFactory.sol:
>```js
>221:         if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
>
>73:             if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
>```

## [G-05] Internal functions only called once can be inlined to save gas
quest-protocol\contracts\QuestFactory.sol:
```js
151      /// @param account_ The account to grant admin and create quest roles
152:     function grantDefaultAdminAndCreateQuestRole(address account_) internal {
153          _grantRole(DEFAULT_ADMIN_ROLE, account_);
```
## [G-06] Multiple address/id mappings can be combined into a single mapping to a struct
If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Depending on the circumstances and sizes of types, can avoid a Gsset per mapping combined. 

>quest-protocol\contracts\RabbitHoleReceipt.sol:
>```js
>30:     mapping(uint => string) public questIdForTokenId;
>34:     mapping(uint => uint) public timestampForTokenId;
>```
>This can be refactored to
>```js
>30:     struct QuestData{
>31:         string questIdForTokenId;
>32:         uint256 timestampForTokenId;
>33:     }
>34:     mapping(uint256 => QuestData) public questData;
>```