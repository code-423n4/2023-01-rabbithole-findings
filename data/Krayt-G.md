USE CLONE FACTORY PATTERN TO DRASTICALLY REDUCE GAS COST ON QUEST DEPLOYEMENT

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

Considering you are going to deploy a high number of the same contracts (ERC20 & ERC1155 Quests) you should use clones inside your factory to cut on the gas costs.
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/proxy/ClonesUpgradeable.sol

Using a basic ERC20 factory deployer vs clone we cut the gas costs of deploying a new contract from 1179977 wei down to 209109 wei which is only 17.7% of the initial cost
The deploy cost for the factory increases as a result of deploying the implementation address from 1245959 wei to 1305848 wei which is a 4.5 % increase.

These gas costs are only examples, the gas cost reduction for the quest contracts is likely higher considering the cost of deployement of those quest contracts.

The only drawback will be a higher gas cost for the factory deployement but will be quickly compensated by the gas cost reduction from the clones