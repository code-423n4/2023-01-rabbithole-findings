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