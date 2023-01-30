Low Crit + non crit :

[L-01] USE SAFETRANSFEROWNERSHIP INSTEAD OF TRANSFEROWNERSHIP FUNCTION:
 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9

transferOwnership function is used to change Ownership from OwnableUpgradeable.sol.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function , use it is more secure due to 2-stage ownership transfer.

Ownable2StepUpgradeable.sol

[N-01] IMPLEMENT SOME TYPE OF VERSION COUNTER THAT WILL BE INCREMENTED AUTOMATICALLY FOR CONTRACT UPGRADES
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L41
I suggest implementing some kind of version counter that will be incremented automatically when you upgrade the contract.

Recommended Mitigation Steps
uint256 public authorizeUpgradeCounter;
 function upgradeTo(address _newImplementation) external onlyOwner {
        _setPendingImplementation(_newImplementation);
       authorizeUpgradeCounter+=1;
    }

[N-02] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USED TO IMMUTABLE RATHER THAN CONSTANT
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

src/QuestFactory.sol:
    19:     bytes32 public constant CREATE_QUEST_ROLE = keccak256("CREATE_QUEST_ROLE");

[N-03] MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L41

If someone wants to extend via inheritance, it might make more sense that the overridden initialize(…) function calls the internal {…}_init function, not the parent public initialize(…) function.

External instead of public would give more the sense of the initialize(…) functions to behave like a constructor (only called on deployment, so should only be called externally)

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

It might cost a bit less gas to use external over public.

It is possible to override a function from external to public (= “opening it up”) :white_check_mark:

But it is not possible to override a function from public to external (= “narrow it down”). :x:

For above reasons you can change initialize(…) to external

https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

[N-04] CHECK FOR ADDRESS != 0

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L175

Should check that the address is != 0 just in case.

[N-05] USE THE SETTER FUNCTION INSTEAD OF CHANGING THE VALUE DIRECTLY

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L47

claimSignerAddress can be set using setClaimSignerAddress line 167 since the initialize function is setting the msg.sender as the owner previously.

[N-06] FLOATING PRAGMA
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List:
pragma ^0.8.15 (all contracts)

Recommendation
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.
