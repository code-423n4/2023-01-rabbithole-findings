G1. There is no need for the ``isClaimed()`` function:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104-L108

We can save gas by check ``claimedList`` directly:
```
uint len = tokens.length;
for (uint i; i < len; unchecked{i++}) {
            if (!claimedList) {
                redeemableTokenCount++;
            }
}
```

G2. https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87
``onlyOwner`` is used for both ``withdrawRemainingTokens()`` and its super method which is also called in ``withdrawRemainingTokens()``:
```
 function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_);
```
One modifier ``onlyOwner`` can be eliminated to save gas here.


