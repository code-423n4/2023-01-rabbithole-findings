## User will not be able to claim tickets

Contract:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159

Impact:
Changing the claimSignerAddress will invalidate all past user tickets.

Proof of Concept:
1. Assume claimSignerAddress was initially address `A`
2. This is used in signature verification so once user tries to mint receipt, it checks whether signer is indeed claimSignerAddress

```
function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
       ...
        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

        ...
    }
```

3. This becomes problem if claimSignerAddress changes as then `mintReceipt` will fail on existing issued signatures due to unmatching claimSignerAddress

Recommended Mitigation Steps:
Do not allow changing claimSignerAddress

## Initialize can be frontrun

Contract:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37

Impact:
The intialize function can currently be front run by attacker and set all initialization parameter. This will force product team to redeploy, wasting gas

Steps:
1. Observe that initialize function can be called by anyone

```
function initialize(
        address claimSignerAddress_,
        address rabbitholeReceiptContract_,
        address protocolFeeRecipient_
    ) public initializer {
        __Ownable_init();
        __AccessControl_init();
        grantDefaultAdminAndCreateQuestRole(msg.sender);
        claimSignerAddress = claimSignerAddress_;
        rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
        setProtocolFeeRecipient(protocolFeeRecipient_);
        setQuestFee(2_000);
        questIdCount = 1;
    }
```

Recommendation:
Use a batch transaction to perform deployment and initialization together

## Fee recipient can be zero address

Contract:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165

Impact:
It was observed that Fee recipient can be mistakenly set as 0 address. This will lead to all fees getting lost in zero address

Steps:
1. Owner mistakenly set protocolFeeRecipient to address 0 using `setProtocolFeeRecipient` function
2. After quest ends, withdrawFee was called by owner
3. All collected fee is sent to protocolFeeRecipient which is address 0
4. Admin calls withdrawFee function which sends all funds to protocolFeeRecipient (0 address)

Recommendation:
Do not allow 0 address protocolFeeRecipient



