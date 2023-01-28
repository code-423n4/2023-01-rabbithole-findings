## Low Vulnerabilities

1. the function claim() is marked as virtual but it isn’t being overriden by any other contracts

2. Renouncing of Ownership is possible in QuestFactory,quests,RabbitHoleReceipt, and RabbitHoleTickets. This can represent a certain risk if the ownership is renounced for any other reason than by design since a lot of functions rely on the owner.

3. In QuestFactory.sol, constructor is being used and the initializer modifier is attached to it. This shouldn’t be done in the case of upgradeable contracts

## Non Critical

1. If TODO is done or implemented, Please remove it from the code. Appears on IQuest.sol

2. uint and uint256 both are being used. Use any one for better code readability

