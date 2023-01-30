`RabbitHoleReceipt.getOwnedTokenIdsOfQuest` is currently O(2n) as implemented and can be optimized as follows -

```
    function getOwnedTokenIdsOfQuestOpt(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory tokenIdsForQuest) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
        tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens;

        uint tokenId;
        bytes32 questHash = keccak256(bytes(questId_));

        unchecked {
          for (uint i; i < msgSenderBalance; i++) {
              tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
              if (tokenId != 0 && keccak256(bytes(questIdForTokenId[tokenId])) == questHash) {
                  tokenIdsForQuest[foundTokens++] = tokenId;
              }
          }
          msgSenderBalance -= foundTokens;
        }

        // truncate array
        assembly { mstore(tokenIdsForQuest, sub(mload(tokenIdsForQuest), msgSenderBalance)) }
    }
```

Gas savings -

```
| Function Name                                              | min             | avg    | median | max    | # calls |
| getOwnedTokenIdsOfQuest                                    | 338955          | 338955 | 338955 | 338955 | 2       |
| getOwnedTokenIdsOfQuestOpt                                 | 256446          | 256446 | 256446 | 256446 | 2       |
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109

---

Use `calldata` instead of `memory` for `data`

```
    function mint(address to_, uint256 id_, uint256 amount_, bytes calldata data_) public onlyMinter {
        _mint(to_, id_, amount_, data_);
    }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L83

```
    function mintBatch(
        address to_,
        uint256[] memory ids_,
        uint256[] memory amounts_,
        bytes calldata data_
    ) public onlyMinter {
        _mintBatch(to_, ids_, amounts_, data_);
    }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L92