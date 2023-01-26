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
