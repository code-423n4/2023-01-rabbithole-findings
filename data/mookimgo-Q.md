# 1. Inconsistency between docs `Create Quest Flow` of README and code

https://github.com/code-423n4/2023-01-rabbithole/blob/main/README.md?plain=1#L56

![image](https://user-images.githubusercontent.com/14314818/214160118-16239aa0-18ca-4963-b812-df4f26153a94.png)

This image shows that **user calls `start()`**, however the code require only owner can call start()

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50

```
function start() public virtual onlyOwner {
```

Fix suggestion: update docs image to clarify that only owner can trigger `start()` function