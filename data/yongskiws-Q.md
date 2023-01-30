 ### [LOW-1] initialize() function can be called by anybody
initialize() function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the  initialize() function. Also, there is no 0 address check in the address arguments of the initialize() function, which must be defined.

Here is a definition of initialize() function.
consider validation check
``` solidity
if (msg.sender != DEPLOYER_ADDRESS) {
         revert NotDeployer();
}
```

``` solidity
quest\QuestFactory.sol
37:     function initialize(
38:         address claimSignerAddress_,
39:         address rabbitholeReceiptContract_,
40:         address protocolFeeRecipient_
41:     ) public initializer {
42:         __Ownable_init();
43:         __AccessControl_init();
44:         grantDefaultAdminAndCreateQuestRole(msg.sender);
45:         claimSignerAddress = claimSignerAddress_;
46:         rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
47:         setProtocolFeeRecipient(protocolFeeRecipient_);
48:         setQuestFee(2_000);
49:         questIdCount = 1;
50:     }

quest\RabbitHoleReceipt.sol
43:     function initialize(
44:         address receiptRenderer_,
45:         address royaltyRecipient_,
46:         address minterAddress_,
47:         uint royaltyFee_
48:     ) public initializer {
49:         __ERC721_init('RabbitHoleReceipt', 'RHR');
50:         __ERC721URIStorage_init();
51:         __Ownable_init();
52:         royaltyRecipient = royaltyRecipient_;
53:         minterAddress = minterAddress_;
54:         royaltyFee = royaltyFee_;
55:         ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
56:     }

quest\RabbitHoleTickets.sol
32:     function initialize(
33:         address ticketRenderer_,
34:         address royaltyRecipient_,
35:         address minterAddress_,
36:         uint royaltyFee_
37:     ) public initializer {
38:         __ERC1155_init('');
39:         __Ownable_init();
40:         __ERC1155Burnable_init();
41:         royaltyRecipient = royaltyRecipient_;
42:         minterAddress = minterAddress_;
43:         royaltyFee = royaltyFee_;
44:         TicketRendererContract = TicketRenderer(ticketRenderer_);
45:     }
```

### [LOW-2]  Lack of zero address check

``` solidity
RabbitHoleReceipt.sol
52:         royaltyRecipient = royaltyRecipient_;
53:         minterAddress = minterAddress_;
54:         royaltyFee = royaltyFee_;

QuestFactory.sol
45:         claimSignerAddress = claimSignerAddress_;
46:         rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
47:         setProtocolFeeRecipient(protocolFeeRecipient_);

RabbitHoleTickets.sol
41:         royaltyRecipient = royaltyRecipient_;
42:         minterAddress = minterAddress_;
43:         royaltyFee = royaltyFee_;

quest\Erc20Quest.sol ,  Erc1155Quest.sol,  Quest.sol
29:       rewardTokenAddress_,
35:       receiptContractAddress_
39:       protocolFeeRecipient = protocolFeeRecipient_;

```

### [N-1] Open TODOs
Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.
``` solidity
IQuest.sol
4: // TODO clean this whole thing up
```
