Files:
Quest.sol
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115
115:     redeemedTokens += redeemableTokenCount;


Description:
x = x + y is cheaper than x += y

Recommendation:
use this:  redeemedTokens =  redeemedTokens + redeemableTokenCount;