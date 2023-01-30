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