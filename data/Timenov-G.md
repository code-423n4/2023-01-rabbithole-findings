# RabbitHole Quest Protocol gas optimization report by Timenov

## Summary
G-01 Using `x += y` costs more gas than using `x = x + y`.
G-02 Using `mapping(uint256 => bool)` costs more than using BitMap.

### [G-01] Using `x += y` costs more gas than using `x = x + y`.
*There is 1 instances of this issue.*

```solidity
File: contracts/Quest.sol

115: redeemedTokens += redeemableTokenCount;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

### [G-02] Using `mapping(uint256 => bool)` costs more than using BitMap.

We can use BitMap instead of `mapping(uint256 => bool)`.

To import it use:

```solidity
import {BitMaps} from "@openzeppelin/contracts/utils/structs/BitMaps.sol";
```

To use it add:

```solidity
using BitMaps for BitMaps.BitMap;
```

To declare it use:
```solidity
BitMaps.BitMap private bitmap;
```

*There is 1 instances of this issue.*

```solidity
File: contracts/Quest.sol

24: mapping(uint256 => bool) private claimedList;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol