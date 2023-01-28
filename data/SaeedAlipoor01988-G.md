Why are you using a constructor when your smart contract is upgradeable and you don’t need a constructor function?
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

Due to a requirement of the proxy-based upgradeability system, no constructors can be used in upgradeable contracts.
https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers
https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#the-constructor-caveat

//////////////////////////////////////////////////////////////////