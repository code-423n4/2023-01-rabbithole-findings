
# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | `royaltyFee` should have a maximum value of `10000`|Low | 1 |
| 2 | Front-runable `initialize` function | Low | 1 |
| 3 | Setter functions should emit an event | NC | 5 |
| 4 | `pause()` and `unPause()` functions can be refactored into a single function | NC | 1 |
| 5 | Named return variables not used anywhere in the functions | NC | 2 |
| 6 | Use scientific notation | NC | 5 |


## Findings

### 1- `royaltyFee` should have a maximum value of 10000 :

In both `RabbitHoleReceipt` and `RabbitHoleTickets` contracts the `royaltyFee` is set in basis point to calculate the fee taken from the sale price, the maximum fee should be 10000 (the denominator in the royaltyInfo function) but the `setRoyaltyFee` function doesn't check the new fee which could be greater than 10000 and in that case funds greater than the actual sale price will be transferred to the fee recepient. 

#### Risk : Low

#### Proof of Concept

Issue occurs in the instance below :

File: RabbitHoleReceipt.sol [Line 90-93](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90-L93)
```
/** @audit
    owner can set royaltyFee to any value
*/
function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
    royaltyFee = royaltyFee_;
    emit RoyaltyFeeSet(royaltyFee_);
}
```

File: RabbitHoleTickets.sol [Line 66-69](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66-L69)
```
/** @audit
    owner can set royaltyFee to any value
*/
function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
    royaltyFee = royaltyFee_;
    emit RoyaltyFeeSet(royaltyFee_);
}
```

As you can see the owner can set any value for `royaltyFee` variables even `type(uint256).max`.

#### Mitigation

To avoid this issue i recommend to add a check in the `setRoyaltyFee` function to ensure that the `royaltyFee` is always less than 10000, the `setRoyaltyFee` function should be modified as follow :
``` 
function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
    if (royaltyFee_ > 10_000) revert RoyaltyFeeTooHigh();
    royaltyFee = royaltyFee_;
    emit RoyaltyFeeSet(royaltyFee_);
}
```

### 2- Front-runable `initialize` function :

The `initialize` function is used in the QuestFactory contracts to initialize important contract parameters (ownership & access control), there is the issue that any attacker can initialize the contract before the legitimate deployer and even if the developers when deploying call immediately the `initialize` function, malicious agents can trace the protocol deployment transactions and insert their own transaction between them and by doing so they front run the developers call and gain the ownership of the contract and set the wrong parameters.

The impact of this issue is : 

* In the best case developers notice the problem and have to redeploy the contract and thus costing more gas.

* In the worst case the protocol continue to work with the wrong owner and state parameters which could lead to the loss of user funds.

#### Risk : Low 

#### Proof of Concept

Instances include:

File: QuestFactory.sol [Line 37-50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37-L50)

```
function initialize(
    address claimSignerAddress_,
    address rabbitholeReceiptContract_,
    address protocolFeeRecipient_
) public
```

#### Mitigation

It's recommended to use the constructor to initialize non-proxied contracts.

For initializing proxy (upgradable) contracts deploy contracts using a factory contract that immediately calls initialize after deployment or make sure to call it immediately after deployment and verify that the transaction succeeded.


### 3- Setter functions should emit an event :

The setter functions which are used to change critical protocol parameters (allow reward tokens, fees,...) should emit an event to allow all users and dapps to detect those changes and act correspondly. 

#### Risk : NON CRITICAL

#### Proof of Concept

The are many setter function which don't emit and event :

File: QuestFactory.sol [Line 179](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179)
```
function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) 
```

File: QuestFactory.sol [Line 186](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186)
```
function setQuestFee(uint256 questFee_) public
```

File: Quest.sol [Line 50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)
```
function start() public virtual onlyOwner
```

File: Quest.sol [Line 57](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)
```
function pause() public onlyOwner onlyStarted
```

File: Quest.sol [Line 63](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)
```
function unPause() public onlyOwner onlyStarted
```

#### Mitigation

Emit an event in the setter functions aforementioned.


### 4- `pause()` and `unPause()` functions can be refactored into a single function :

#### Risk : NON CRITICAL

The `pause()` and `unPause()` functions from the Quest contract can be refactored into a single function `setPause` which can help reduce contract code size :

```
function setPause(bool _pause) public onlyOwner onlyStarted {
    if(isPaused != _pause){
        isPaused = _pause;
    }
}
```

### 5- Named return variables not used anywhere in the function :

When Named return variable are declared they should be used inside the function instead of the return statement or if not they should be removed to avoid confusion.

#### Risk : NON CRITICAL

#### Proof of Concept

Instances include:

File: RabbitHoleReceipt.sol [Line 181](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L181)
```
returns (address receiver, uint256 royaltyAmount)
```

File: RabbitHoleTickets.sol [Line 112](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L112)
```
returns (address receiver, uint256 royaltyAmount)
```

#### Mitigation

Either use the named return variables inplace of the return statement or remove them.

### 6- Use scientific notation :

When using multiples of 10 you shouldn't use decimal literals or exponentiation (e.g. 1000000, 10**18) but use scientific notation for better readability.

#### Risk : NON CRITICAL

#### Proof of Concept

Instances include:

File: QuestFactory.sol [Line 187](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187)

```
if (questFee_ > 10_000)
```

File: Erc20Quest.sol [Line 53](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L53)

```
return (maxTotalRewards() * questFee) / 10_000;
```

File: Erc20Quest.sol [Line 97](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L97)

```
return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```

File: RabbitHoleReceipt.sol [Line 184](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184)

```
uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

File: RabbitHoleTickets.sol [Line 113](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113)

```
uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

#### Mitigation
Use scientific notation for the instances aforementioned.
