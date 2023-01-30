## Summary

Some deployment gas costs can be saved by sorting the storage variables in a way that packs well in 256 bit/32 byte slots, as well as by not initializing storage variables to their default values.

## Findings

- **Storage Variables Order** - deployment gas costs can be saved by sorting the storage variables in a way that packs well in 256 bit/32 byte slots:

|`quest-protocol/contracts/Quest.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L13|

- **Useless Initialization** - deployment gas costs can be saved by not initializing storage variables to their default values:

|`quest-protocol/contracts/Quest.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45|
