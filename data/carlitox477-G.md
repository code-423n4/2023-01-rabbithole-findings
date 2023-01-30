# Quest.constructor: Remove ```endTime_ > block.timestamp``` check and EndTimeInPast error declaraion to save gas
Currently

```solidity
// Quest constructor
    if (endTime_ <= block.timestamp) revert EndTimeInPast();
    if (startTime_ <= block.timestamp) revert StartTimeInPast();
    if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
```
This means thats next checks are done:
1. ```endTime_ > block.timestamp```
2. ```startTime_ > block.timestampp```
3. ```endTime_ > startTime_```

From 2 and 3: ```endTime_ > startTime_ > block.timestampp```
Then 1 can be omitted, saving gas. Line ```if (endTime_ <= block.timestamp) revert EndTimeInPast();``` should be removed, as well as ```EndTimeInPast()``` error declaration.



# ```Erc20Quest.maxTotalRewards``` and ```Erc20Quest.maxProtocolReward``` can be immutable in order to save gas
Given than ```totalParticipants```, ```rewardAmountInWeiOrTokenId``` and ```questFee``` are immutable, and current function implementations, these values can calculated during construction to save gas each time they are used inside the contract. Then:
1. Remove ```Erc20Quest.maxTotalRewards``` and ```Erc20Quest.maxProtocolReward```
2. Add immutable variables and modify constructor:
```diff
// Erc20Quest.sol
+   uint256 public immutable maxTotalRewards;
+   uint256 public immutable maxProtocolReward;
+   uint256 internal immutable sumMaxRewards;
    constructor(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountInWeiOrTokenId_,
        string memory questId_,
        address receiptContractAddress_,
        uint256 questFee_,
        address protocolFeeRecipient_
    )
        Quest(
            rewardTokenAddress_,
            endTime_,
            startTime_,
            totalParticipants_,
            rewardAmountInWeiOrTokenId_,
            questId_,
            receiptContractAddress_
        )
    {
        questFee = questFee_;
        protocolFeeRecipient = protocolFeeRecipient_;
        questFactoryContract = QuestFactory(msg.sender);
+       maxTotalRewards = totalParticipants_ * rewardAmountInWeiOrTokenId;
+       maxProtocolReward = (maxTotalRewards * questFee) / 10_000;
+       sumMaxRewards = maxTotalRewards + maxProtocolReward;
    }
```
3. Modify internal dependancies:
```diff
// Erc20Quest.sol
    function start() public override {
-       if (IERC20(rewardToken).balanceOf(address(this)) < maxTotalRewards() + maxProtocolReward())
+       if (IERC20(rewardToken).balanceOf(address(this)) < sumMaxRewards)
            revert TotalAmountExceedsBalance();
        super.start();
    }
```

This would also prevent any possible overflow and it consequences, given that constract creation will revert if this is the case.

# Do not check if bool value is true
This can be avoided by just using the bool value:
```diff
// Quest.isClaimed
-   return claimedList[tokenId_] == true;
+   return claimedList[tokenId_];

// QuestFactory.mintReceipt
-   if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
+   if (quests[questId_].addressMinted[msg.sender]) revert AddressAlreadyMinted();
```

# Make Function external instead of public
```generateTokenURI``` is just used by external contracts, then it visibility should be change to external in ```ReceiptRender``` as well as in ```TicketRender``` to save gas

Then, next function are never used internally:
```diff
// Quest
-   function pause() public onlyOwner onlyStarted {
+   function pause() external onlyOwner onlyStarted {
//...

-   function unPause() public onlyOwner onlyStarted {
+   function unPause() external onlyOwner onlyStarted {
//...

-   function claim() public virtual onlyQuestActive {
+   function claim() external virtual onlyQuestActive {
//...

-   function getRewardAmount() public view returns (uint256) {
+   function getRewardAmount() external view returns (uint256) {
//...

-   function getRewardToken() public view returns (address) {
+   function getRewardToken() external view returns (address) {
//...

// Erc20Quest
-   function start() public override {
+   function start() external override {
//...

// RabbitHoleTickets.sol
    function mintBatch(
        address to_,
        uint256[] memory ids_,
        uint256[] memory amounts_,
        bytes memory data_
-   ) public onlyMinter {
+   ) external onlyMinter {
//...

-   function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
+   function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) external onlyMinter {
//...

-   function setMinterAddress(address minterAddress_) public onlyOwner {
+   function setMinterAddress(address minterAddress_) external onlyOwner {
//...

-   function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
+   function setRoyaltyFee(uint256 royaltyFee_) external onlyOwner {
//...

-   function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
+   function setRoyaltyRecipient(address royaltyRecipient_) external onlyOwner {
//...

-   function setTicketRenderer(address ticketRenderer_) public onlyOwner {
+   function setTicketRenderer(address ticketRenderer_) external onlyOwner {
//...
```

# ```QuestFactory.createQuest``` can be refactor to save gas during deployment
Next lines are repeated twice in the function
```solidity
quests[questId_].questAddress = address(newQuest);
quests[questId_].totalParticipants = totalParticipants_;
newQuest.transferOwnership(msg.sender);
++questIdCount;
return address(newQuest);
```

This can be avoid by refactoring the code:
```diff
    function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
        // require(quests[questId_].questAddress  != 0)
        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();

        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
            // require(rewardAllowlist[rewardTokenAddress_])
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
-           quests[questId_].questAddress = address(newQuest);
-           quests[questId_].totalParticipants = totalParticipants_;
-           newQuest.transferOwnership(msg.sender);
-           ++questIdCount;
-           return address(newQuest);
        }

-       if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
+       else if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
            // require(msg.sender == owner())
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
-           quests[questId_].questAddress = address(newQuest);
-           quests[questId_].totalParticipants = totalParticipants_;
-           newQuest.transferOwnership(msg.sender);
-           ++questIdCount;
-           return address(newQuest);
        }
+       else{
+           revert QuestTypeInvalid();
+       }
-       revert QuestTypeInvalid();
+       quests[questId_].questAddress = address(newQuest);
+       quests[questId_].totalParticipants = totalParticipants_;
+       newQuest.transferOwnership(msg.sender);
+       unchecked{++questIdCount;}
+       return address(newQuest);
    }
```

# Use storage pointer to save gas in for loops
```diff
    // Quest
    function _setClaimed(uint256[] memory tokenIds_) private {
+       mapping(address =>bool) storage _claimedList = claimedList;
        for (uint i = 0; i < tokenIds_.length; i++) {
-           claimedList[tokenIds_[i]] = true;
+           _claimedList[tokenIds_[i]] = true;
        }
    }

    // RabbitHoleReceipt.getOwnedTokenIdsOfQuest(string memory, address)
+   mapping(uint => string) _questIdForTokenId = questIdForTokenId;
    for (uint i = 0; i < msgSenderBalance; i++) {
        uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
-       if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
+       if (keccak256(bytes(_questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
            tokenIdsForQuest[i] = tokenId;
            foundTokens++;
        }
    }
```

In the case of ```claim``` function in ```Quest``` contract, function ```isClaimed``` is called inside the loop, which make use of ```claimedList``` state variable. In order to save gas using a storage pointer, the loop should be refactored to:

```diff
    // Quest.claim
    uint256 redeemableTokenCount = 0;
+   mapping(address =>bool) storage _claimedList = claimedList;
    for (uint i = 0; i < tokens.length; i++) {
-       if (!isClaimed(tokens[i])) {
+       if (!_claimedList(tokens[i])) {
            redeemableTokenCount++;
        }
    }
```

## Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```diff
//Erc20Quest
    constructor(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountInWeiOrTokenId_,
        string memory questId_,
        address receiptContractAddress_,
        uint256 questFee_,
        address protocolFeeRecipient_
    )
    Quest(
        rewardTokenAddress_,
        endTime_,
        startTime_,
        totalParticipants_,
        rewardAmountInWeiOrTokenId_,
        questId_,
        receiptContractAddress_
-   )
+   ) payable
    //...
-   function start() public override {
+   function start() public payable override {
    //...

-   function withdrawRemainingTokens(address to_) public override onlyOwner {
+   function withdrawRemainingTokens(address to_) public payable override onlyOwner {
    //...

// Erc1155Quest
    constructor(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountInWeiOrTokenId_,
        string memory questId_,
        address receiptContractAddress_
    )
    Quest(
        rewardTokenAddress_,
        endTime_,
        startTime_,
        totalParticipants_,
        rewardAmountInWeiOrTokenId_,
        questId_,
        receiptContractAddress_
-   ){}
+   ) payable {}
    //...

-   function withdrawRemainingTokens(address to_) public override onlyOwner {
+   function withdrawRemainingTokens(address to_) public payable override onlyOwner
    //...

-   function start() public override {
+   function start() public payable override {
    //...

// Quest
    constructor(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountInWeiOrTokenId_,
        string memory questId_,
        address receiptContractAddress_
-   ) {
+   ) payable {
    //...

-   function start() public virtual onlyOwner {
+   function start() public virtual payable onlyOwner {
    //...

-   function pause() public onlyOwner onlyStarted {
+   function pause() public payable onlyOwner onlyStarted {
    //...

-   function unPause() public onlyOwner onlyStarted {
+   function unPause() public payable onlyOwner onlyStarted {

-   function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
+   function withdrawRemainingTokens(address to_) public payable virtual onlyOwner onlyAdminWithdrawAfterEnd {}
    //...

// QuestFactory
-   constructor() initializer {}
+   constructor() payable initializer {}
    //...

    function initialize(
        address claimSignerAddress_,
        address rabbitholeReceiptContract_,
        address protocolFeeRecipient_
-   ) public initializer {
+   ) public payable initializer {
    //...

    function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
-   ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
+   ) public payable onlyRole(CREATE_QUEST_ROLE) returns (address) {
    //...

-   function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
+   function changeCreateQuestRole(address account_, bool canCreateQuest_) public payable onlyOwner {
    //...

-   function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
+   function setClaimSignerAddress(address claimSignerAddress_) public payable onlyOwner {
    //...

-   function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
+   function setProtocolFeeRecipient(address protocolFeeRecipient_) public payable onlyOwner {
    //...

-   function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
+   function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public payable onlyOwner {
    //...

-   function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
+   function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public payable onlyOwner {
    //...

-   function setQuestFee(uint256 questFee_) public onlyOwner {
+   function setQuestFee(uint256 questFee_) public payable onlyOwner { 
    //..
```