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


for the code
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

1. Use the ‘view’ keyword in functions that don’t modify the state of the contract. This will reduce the amount of gas consumed. 
```
function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
    return TicketRendererContract.generateTokenURI(tokenId_);
}
```
2. Avoid reading from multiple locations in memory. This will reduce the amount of gas consumed. 
``` 
function royaltyInfo(
    uint256 tokenId_,
    uint256 salePrice_
) external view override returns (address receiver, uint256 royaltyAmount) {
    uint256 royaltyPayment = salePrice_ * royaltyFee / 10_000;
    return (royaltyRecipient, royaltyPayment);
}
```
3. Use the ‘pure’ keyword instead of ‘view’ when possible. This will reduce the amount of gas consumed. 
``` 
function supportsInterface(
    bytes4 interfaceId_
) public pure virtual override(ERC1155Upgradeable, IERC165Upgradeable) returns (bool) {
    return interfaceId_ == type(IERC2981Upgradeable).interfaceId || super.supportsInterface(interfaceId_);
}
```
4. Use assembly code to optimize loops. This will reduce the amount of gas consumed. 
``` 
pragma solidity ^0.8.15;

contract MyContract {
    function myFunction() public {
        uint256 i;
        for(i=0; i<10; i++) {
            assembly {
                // assembly code here
            }
        }
    }
}
```
5. Use the ‘memory’ keyword to store data in memory instead of storage. This will reduce the amount of gas consumed. 

```
function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
    _mint(to_, id_, amount_, data_);
}
```
6. Use the ‘internal’ keyword to restrict visibility of functions, variables and modifiers to the contract itself. This will reduce the amount of gas consumed. 
``` 
modifier onlyMinter() internal {
    msg.sender == minterAddress;
    _;
}
```
7. Use the ‘constant’ keyword to optimize the calculation of return values. This will reduce the amount of gas consumed. 

For example:
``` 
function supportsInterface(
    bytes4 interfaceId_
) public constant virtual override(ERC1155Upgradeable, IERC165Upgradeable) returns (bool) {
    return interfaceId_ == type(IERC2981Upgradeable).interfaceId || super.supportsInterface(interfaceId_);
}
```
8. Use the ‘staticcall’ keyword to call functions of other contracts without consuming gas. This will reduce the amount of gas consumed. 

For example:
``` 
function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
    return TicketRendererContract.staticcall(bytes4(keccak256("generateTokenURI(uint256)")), tokenId_);
}
```
9. Use the ‘inline assembly’ keyword to optimize operations. This will reduce the amount of gas consumed. 

For example:
``` 
function royaltyInfo(
    uint256 tokenId_,
    uint256 salePrice_
) external view override returns (address receiver, uint256 royaltyAmount) {
    uint256 royaltyPayment;
    assembly {
        royaltyPayment := mul(royaltyFee, div(salePrice_, 10_000))
    }
    return (royaltyRecipient, royaltyPayment);
}
```