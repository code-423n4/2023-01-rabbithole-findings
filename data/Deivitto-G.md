
# Gas

## Multiple `address`/`ID` `mappings` can be combined into a single `mapping` of an `address`/`ID` to a `struct`, where appropriate

if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s `keccak256` hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

Both mapping being used in the same functions mostly consider making them a struct instead 


- [QuestFactory.sol#L20](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L20)
        mapping(address => bool) addressMinted;

- [QuestFactory.sol#L30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L30)
    mapping(address => bool) public rewardAllowlist;

- [RabbitHoleReceipt.sol#L30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L30)
    mapping(uint => string) public questIdForTokenId;

- [RabbitHoleReceipt.sol#L34](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L34)
    mapping(uint => uint) public timestampForTokenId;


## state variables can be packed into fewer storage slots
If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

consider making this state var near one of the address types

- [Quest.sol#L14](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L14)
    address public immutable rewardToken;

- [Quest.sol#L19](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L19)
    bool public hasStarted;

- [Quest.sol#L20](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L20)
    bool public isPaused;


## `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas


- [Quest.sol#L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L115)
        redeemedTokens += redeemableTokenCount;



## Not using the named return variables when a function returns, wastes deployment gas

- [RabbitHoleReceipt.sol#L181](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L181)
    ) external view override returns (address receiver, uint256 royaltyAmount) {

- [RabbitHoleTickets.sol#L112](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleTickets.sol#L112)
    ) external view override returns (address receiver, uint256 royaltyAmount) {

## `++i / i++` should be `unchecked{++i} / unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

- [RabbitHoleReceipt.sol#L117](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L117)
        for (uint i = 0; i < msgSenderBalance; i++) {

- [RabbitHoleReceipt.sol#L128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L128)
        for (uint i = 0; i < msgSenderBalance; i++) {

- [Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L70)
        for (uint i = 0; i < tokenIds_.length; i++) {

- [Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L104)
        for (uint i = 0; i < tokens.length; i++) {



## `internal` functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

- [QuestFactory.sol#L152](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L152)
    function grantDefaultAdminAndCreateQuestRole(address account_) internal {

- [RabbitHoleReceipt.sol#L139](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L139)
    function _burn(uint256 tokenId_) internal override(ERC721Upgradeable, ERC721URIStorageUpgradeable) {

- [Quest.sol#L122](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L122)
    function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {

- [Quest.sol#L129](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L129)
    function _transferRewards(uint256 amount_) internal virtual {

- [Erc20Quest.sol#L66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Erc20Quest.sol#L66)
    function _transferRewards(uint256 amount_) internal override {

- [Erc20Quest.sol#L74](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Erc20Quest.sol#L74)
    function _calculateRewards(uint256 redeemableTokenCount_) internal view override returns (uint256) {

- [Erc1155Quest.sol#L41](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Erc1155Quest.sol#L41)
    function _transferRewards(uint256 amount_) internal override {

- [Erc1155Quest.sol#L48](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Erc1155Quest.sol#L48)
    function _calculateRewards(uint256 redeemableTokenCount_) internal pure override returns (uint256) {

- [RabbitHoleReceipt.sol#L153](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L153)
    ) internal override(ERC721Upgradeable, ERC721EnumerableUpgradeable) {







## Extra check on booleans

There are instances where a boolean variable / function is checked against a boolean unnecessarily wasting extra ~18 gas for instance

- Checks can be simplified from `variable == false` to `!variable`.
- Checks can be simplified from `variable == true` to `variable`.

- [QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L221)
        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

- [Quest.sol#L136](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/Quest.sol#L136)
        return claimedList[tokenId_] == true;

- [QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/QuestFactory.sol#L73)
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();





## Use Custom Errors

`NOTE`: None of these findings where found by [4naly3er output - Gas](https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480)

[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/) Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

- [RabbitHoleReceipt.sol#L161](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L161)
        require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');

- [RabbitHoleReceipt.sol#L162](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L162)
        require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');

- [RabbitHoleReceipt.sol#L182](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d//contracts/RabbitHoleReceipt.sol#L182)
        require(_exists(tokenId_), 'Nonexistent token');




