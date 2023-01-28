### [G-01] Use named returns where appropriate

**contracts\RabbitHoleReceipt.sol**

    112: ) public view returns (uint[] memory) {

**contracts\ReceiptRenderer.sol**

    47: ) public view virtual returns (bytes memory) {


### [G-02] X = X + Y is more efficient, than X += Y 

**contracts\Quest.sol**

    115: redeemedTokens += redeemableTokenCount;

### [G-03]  keccak256(abi.encodePacked('erc1155')) and keccak256(abi.encodePacked('erc20')) could be computed in advance and saved as immutable to save some gas

**contracts\QuestFactory.sol**

Every time calling function createQuest() we need to spend some gas to calculate keccak256(abi.encodePacked('erc1155') and keccak256(abi.encodePacked('erc20')). If we save them as immutable variables their values will be computed only once deploying a contract.

    - 72:  if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20')))
    - 105: if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155')))

    + bytes32 immutable CONTRACT_TYPE_ERC20 = keccak256(abi.encodePacked('erc20'));
    + bytes32 immutable CONTRACT_TYPE_ERC1155 = keccak256(abi.encodePacked('erc1155'));
    + if (keccak256(abi.encodePacked(contractType_)) == CONTRACT_TYPE_ERC20)
    + if (keccak256(abi.encodePacked(contractType_)) == CONTRACT_TYPE_ERC1155)
