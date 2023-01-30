for code https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol
1. Use view or pure functions when possible:
```
function getNumberMinted(string memory questId_) external view returns (uint) {
        return quests[questId_].numberMinted;
    }
```
2. Avoiding unnecessary computations:
```
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
```
3. Pre-compute values that are repeated multiple times in the same scope:
```
bytes32 hashType20 = keccak256(abi.encodePacked('erc20'));
bytes32 hashType1155 = keccak256(abi.encodePacked('erc1155'));

if (keccak256(abi.encodePacked(contractType_)) == hashType20) {
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
```
4. Reduce the number of parameters by placing multiple parameters in a struct:
```
struct QuestCreationParams {
    address rewardTokenAddress;
    uint256 endTime;
    uint256 startTime;
    uint256 totalParticipants;
    uint256 rewardAmountOrTokenId;
    string contractType;
    string questId;
}

function createQuest(QuestCreationParams params) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
```
5. Avoiding nested if-statements:
```
bytes32 hashType20 = keccak256(abi.encodePacked('erc20'));
bytes32 hashType1155 = keccak256(abi.encodePacked('erc1155'));

if (keccak256(abi.encodePacked(params.contractType)) == hashType20) {
            if (rewardAllowlist[params.rewardTokenAddress] == false) revert RewardNotAllowed();
            Erc20Quest newQuest = new Erc20Quest(
                params.rewardTokenAddress,
                params.endTime,
                params.startTime,
                params.totalParticipants,
                params.rewardAmountOrTokenId,
                params.questId,
                address(rabbitholeReceiptContract),
                questFee,
                protocolFeeRecipient
            );
            emit QuestCreated(
                msg.sender,
                address(newQuest),
                params.questId,
                params.contractType,
                params.rewardTokenAddress,
                params.endTime,
                params.startTime,
                params.totalParticipants,
                params.rewardAmountOrTokenId
            );
            quests[params.questId].questAddress = address(newQuest);
            quests[params.questId].totalParticipants = params.totalParticipants;
            newQuest.transferOwnership(msg.sender);
            ++questIdCount;
            return address(newQuest);
} else if (keccak256(abi.encodePacked(params.contractType)) == hashType1155) {
            if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest();
            Erc1155Quest newQuest = new Erc1155Quest(
                params.rewardTokenAddress,
                params.endTime,
                params.startTime,
                params.totalParticipants,
                params.rewardAmountOrTokenId,
                params.questId,
                address(rabbitholeReceiptContract)
            );
            emit QuestCreated(
                msg.sender,
                address(newQuest),
                params.questId,
                params.contractType,
                params.rewardTokenAddress,
                params.endTime,
                params.startTime,
                params.totalParticipants,
                params.rewardAmountOrTokenId
            );
            quests[params.questId].questAddress = address(newQuest);
            quests[params.questId].totalParticipants = params.totalParticipants;
            newQuest.transferOwnership(msg.sender);
            ++questIdCount;
            return address(newQuest);
}
```