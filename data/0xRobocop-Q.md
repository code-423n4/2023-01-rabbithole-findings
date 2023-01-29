## [L-01] Use ``Ownable2StepUpgradeable`` and ``Ownable2Step`` instead of ``OwnableUpgradeable`` and ``Ownable``.

Description:

Using a 1-step transfer of ownership is dangerous. If an incorrect address is passed as the ``new owner``, there is no coming back.

Better use the 2-step pattern from OZ. [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) and [Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol).

Found at :

[Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol)
[QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)
[RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)

## [L-02] Use a nonReentrant mutex to avoid reentrancy issues.

Description:

When using _safeMint() of the ERC721 OZ implementation there is a call to the address of the ``_to`` paramenter to make sure the account can receive an ERC-721 token. This can be used to re-enter the contract causing unexpected behaviors.

This can happen also when using erc-777 tokens instead of erc-20 tokens. 

Use a mutex like the OZ [nonReentrant](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol#L50)

Found at:

[RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L103).

## [L-03] The claim function might use an amount of gas greater than the block gas limit.

Description:

The [claim](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96) function at the Quest.sol contract can consume an amount of gas greater than the block gas limit if the user has too many receipts.

Mitigation:

Make users know they might need to split their receipts accross other wallets in this case.

## [L-04] Use EIP-712 for signatures.

For a better usability of off-chain message signing for use on-chain use the [EIP-712](https://eips.ethereum.org/EIPS/eip-712) standard.

## [L-05] ERC-20 Tokens stucked if protocolFeeReceipt cannot handle them.

There is a non-zero risk of making a mistake of setting ``protocolFeeReceipt`` address to an account not been able to manage ERC-20 tokens. Effectively losing all the tokens send to it through the protocol fee.


