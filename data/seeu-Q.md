## [L-01] For modern and more readable code; update import usages

### Description

A less obvious way that solidity code is clearer is the struct Point. Prior to now, we imported it via global import, but we didn't use it. The Point struct contaminated the source code with an extra object that was not needed and that we were not utilizing.

To be sure to only import what you need, use specific imports using curly brackets.

### Findings

- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol
  ```Solidity
  ::4 => import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
  ::10 => import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
  ::11 => import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol
  ```Solidity
  ::4 => import '@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol';
  ::5 => import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721URIStorageUpgradeable.sol';
  ::6 => import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';
  ::7 => import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
  ::8 => import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
  ::9 => import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
  ::10 => import '@openzeppelin/contracts-upgradeable/utils/CountersUpgradeable.sol';
  ::11 => import './ReceiptRenderer.sol';
  ::12 => import './interfaces/IQuestFactory.sol';
  ::13 => import './interfaces/IQuest.sol';
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol
  ```Solidity
  ::4 => import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
  ::5 => import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
  ::6 => import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';
  ::7 => import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
  ::8 => import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
  ::9 => import './TicketRenderer.sol';
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol
  ```Solidity
  ::4 => import '@openzeppelin/contracts/utils/Base64.sol';
  ::5 => import '@openzeppelin/contracts/utils/Strings.sol';
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol
  ```Solidity
  ::4 => import '@openzeppelin/contracts/utils/Base64.sol';
  ::5 => import '@openzeppelin/contracts/utils/Strings.sol';
  ```

### Resources

- [[N-03] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES | PoolTogether contest](https://code4rena.com/reports/2022-12-pooltogether#n-03-for-modern-and-more-readable-code-update-import-usages)


## [L-02] Unnamed return parameters

### Description

To increase explicitness and readability, take into account introducing and utilizing named return parameters.

### Findings

- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol
  ```Solidity
  ::48 =>     function _calculateRewards(uint256 redeemableTokenCount_) internal pure override returns (uint256) {
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol
  ```Solidity
  ::45 =>     function maxTotalRewards() public view returns (uint256) {
  ::52 =>     function maxProtocolReward() public view returns (uint256) {
  ::74 =>     function _calculateRewards(uint256 redeemableTokenCount_) internal view override returns (uint256) {
  ::91 =>     function receiptRedeemers() public view returns (uint256) {
  ::96 =>     function protocolFee() public view returns (uint256) {
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol
  ```Solidity
  ::122 =>     function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
  ::135 =>     function isClaimed(uint256 tokenId_) public view returns (bool) {
  ::140 =>     function getRewardAmount() public view returns (uint256) {
  ::145 =>     function getRewardToken() public view returns (address) {
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol
  ```Solidity
  ::193 =>     function getNumberMinted(string memory questId_) external view returns (uint) {
  ::199 =>     function questInfo(string memory questId_) external view returns (address, uint, uint) {
  ::210 =>     function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol
  ```Solidity
  ::102 =>     function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol
  ```Solidity
  ::82 =>     function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
  ::100 =>     function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
  ```
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol
  ```Solidity
  ::36 =>     function generateSVG(uint tokenId_) public pure returns (string memory) {
  ```

### References

- [Unnamed return parameters | Opyn Bull Strategy Contracts Audit](https://blog.openzeppelin.com/opyn-bull-strategy-contracts-audit/#unnamed-return-parameters)


## [L-03] Compiler version Pragma is non-specific

### Description

For non-library contracts, floating pragmas may be a security risk for application implementations, since a known vulnerable compiler version may accidentally be selected or security tools might fallback to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

### Findings

- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol
  => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol
  => `pragma solidity ^0.8.15;`

### References

- [L003 - Unspecific Compiler Version Pragma](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)
- [Version Pragma | Solidity documents](https://docs.soliditylang.org/en/latest/layout-of-source-files.html#version-pragma)
- [4.6 Unspecific compiler version pragma | Consensys Audit of 1inch Liquidity Protocol](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unspecific-compiler-version-pragma)


## [L-04] Outdated Compiler Version

### Description

Using an older compiler version might be risky, especially if the version in question has faults and problems that have been made public.

### Findings

- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol => `pragma solidity ^0.8.15;`
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol => `pragma solidity ^0.8.15;`

### Resources

- [SWC-102](https://swcregistry.io/docs/SWC-102)
- [Etherscan Solidity Bug Info](https://etherscan.io/solcbuginfo)


## [L-05] Use of abi.encodePacked instead of bytes.concat() for Solidity version >= 0.8.4

### Description

From the Solidity version `0.8.4` it was added the possibility to use `bytes.concat` with variable number of `bytes` and `bytesNN` arguments. With a more evocative name, it functions as a restricted `abi.encodePacked`.

### Findings

- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol => `pragma solidity ^0.8.15;`
  ```Solidity
  ::72 =>         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
  ::105 =>         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
  ::211 =>         bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));
  ::222 =>         if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
  ```
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol => `pragma solidity ^0.8.15;`
  ```Solidity
  ::37 =>         return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));
  ::48 =>         bytes memory attributes = abi.encodePacked(
  ::63 =>         bytes memory dataURI = abi.encodePacked(
  ::83 =>         bytes memory attribute = abi.encodePacked(
  ::101 =>         bytes memory svg = abi.encodePacked(
  ::113 =>         return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
  ```
- https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol => `pragma solidity ^0.8.15;`
  ```Solidity
  ::19 =>         bytes memory dataURI = abi.encodePacked(
  ::30 =>         return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));
  ::37 =>         bytes memory svg = abi.encodePacked(
  ::46 =>         return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
  ```

### Resources

- [Solidity 0.8.4 Release Announcement](https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/)
- [Remove abi.encodePacked #11593](https://github.com/ethereum/solidity/issues/11593)