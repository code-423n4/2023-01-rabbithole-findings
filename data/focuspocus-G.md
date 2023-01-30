for code https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

1. Avoid using the keccak256() function when possible, as it consumes a lot of gas. For example, by creating a constant to store the keccak256 of a string instead of only keccak256() it. 

bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');

2. Avoid using the abi.encodePacked() function when possible, as it consumes a lot of gas. For example, by creating a constant to store the abi.encodePacked of a string instead of only abi.encodePacked() it. 

bytes32 public constant ERC20_QUEST = keccak256(abi.encodePacked('erc20'));

3. Reorder the function parameters in the most efficient way, so that the more expensive computations are last.

function createQuest(
        string memory questId_,
        string memory contractType_,
        address rewardTokenAddress_,
        uint256 startTime_,
        uint256 endTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_
    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {

4. Use the most efficient data types when possible. For example, use uint8 instead of uint256 if the value will never go above 255.

uint8 public questIdCount;

5. Use the most efficient loop type when possible. For example, use for-loops instead of while-loops.

for (uint256 i = 0; i < totalParticipants_; i++) {

6. Use the most efficient comparison operator when possible. For example, use == instead of != whenever possible.

if (quests[questId_].questAddress == address(0)) revert QuestIdUsed();

7. Avoid using the revert() function when possible, as it consumes a lot of gas. For example, by using an if-else statement instead.

if (questFee_ > 10_000) revert QuestFeeTooHigh();
else questFee = questFee_;

8. Use the most efficient struct type when possible. For example, use a mapping instead of a struct whenever possible.

mapping(address => bool) public rewardAllowlist;

9. Use the most efficient arrays type when possible. For example, use bytes32 instead of string when possible.

bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');

10. Avoid unnecessary calls to external functions when possible. For example, by storing the result of a function call in a variable.

bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));
return ECDSAUpgradeable.recover(messageDigest, signature_);