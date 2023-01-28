### [G-01] Use Immutable instead of constant for `keccak256`

Constant expressions are [re-calculated each time it is in use]([https://github.com/ethereum/solidity/issues/9232](https://github.com/ethereum/solidity/issues/9232)), costing an extra `97` gas than a constant every time they are called.

Mark these as  `immutable`  instead of  `constant`

*There are 1  instances of this issue:*

**Affected source code:**

```solidity
17:  bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```

- [QuestFactory.sol:17](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17)

### [G-02] State variables only set in the constructor should be declared `IMMUTABLE`

Avoids a Gsset (**20000 gas**) in the constructor, and replaces the first access in each transaction (Gcoldsload - **2100 gas**) and each access thereafter (Gwarmacces - **100 gas**) with a `PUSH32` (**3 gas**).

*There are 1  instances of this issue:*

**Affected source code:**

```solidity
21: string public questId;
```

- [Quest.sol:21](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L21)

### [G-03] Comparing to a constant (`true` or `false`) is a bit more expensive

Comparing to a constant (`true` or `false`) is a bit more expensive than directly checking the returned boolean value.I suggest using `if(directValue)` instead of `if(directValue == true)` and `if(!directValue)` instead of `if(directValue == false)` 

*There are 1 instances of this issue:*

**Affected source code:**

- [QuestFactory.sol:73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73)

### [G-04] Help the optimizer by declaring a storage variable instead of repeatedly fetching the value

To help the optimizer, go from:

```solidity
220: if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
221:        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
222:        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
223:    if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();
224:
225:    quests[questId_].addressMinted[msg.sender] = true;
226:    quests[questId_].numberMinted++;
```

to

```solidity
220:	     bool storage _checkAddress = quests[questId_].addressMinted[msg.sender];
221:       uint storage _checkMax	= quests[questId_].numberMinted;
222:        if ( _checkMax + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
223:        if ( _checkAddress == true) revert AddressAlreadyMinted();
224:        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
225:        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();
226:
227:    _checkAddress = true;
228:  unchecked {  
229:       _checkMax++;
230:       }
```

and mark _checkMax unchecked

### [G-05] X = X + Y costs less gass than X += Y

X = X + Y costs less gass than X += Y for state variables

*There are 1 instances of this issue:*

**Affected source code:**

```solidity
/Quest.sol
115: redeemedTokens += redeemableTokenCount; 
```