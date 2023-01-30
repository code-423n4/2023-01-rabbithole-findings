# GAS Report 

## Duplicated `onlyOwner` modifer on `withdrawRemainingTokens` 

The inherited function `withdrawRemainingTokens` in `Quest.sol` already has the `onlyOwner` modifier which is run during `super.withdrawRemainingTokens(to_)`. Remove `onlyOwner` on `withdrawRemainingTokens` in `Erc20Quest`.


[1. File: Erc20Quest.sol#L81](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81)


``` solidity 

function withdrawRemainingTokens(address to_) public override onlyOwner {
    super.withdrawRemainingTokens(to_);
```

## Don't initialize variables with default value

`IsPaused` can never be true when calling `start()` so setting it to its initial value wastes gas. This was missed by the automated findings.

[1. File: Quest.sol#L50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)


``` solidity 

function start() public virtual onlyOwner {
    isPaused = false;

```