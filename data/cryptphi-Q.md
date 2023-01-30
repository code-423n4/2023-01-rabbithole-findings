1. Missing Zero Address Check

The following are missing zero address checks: 
A. QuestFactory.initialize() - `claimSignerAddress_`  argument

B. . RabbitHoleReceipt.initialize()  - `receiptRenderer_` , `royaltyRecipient_` 
C. RabbitHoleReceipt.setReceiptRenderer() - `receiptRenderer_`
This may cause RabbitHoleReceipt.tokenURI() to always return empty string when `ReceiptRendererContract` is set to zero address

D. RabbitHoleReceipt.setRoyaltyRecipient() - `royaltyRecipient_` argument
This may cause a loss in royalty fees when `royaltyRecipient` is set to zero address

E. RabbitHoleTickets.initialize()  - `ticketRenderer_` , `royaltyRecipient_` 
F. RabbitHoleTickets.setReceiptRenderer() - `receiptRenderer_`
This may cause RabbitHoleTickets.uri() to always return empty string when `TicketRendererContract` is set to zero address

G. RabbitHoleTickets.setRoyaltyRecipient() - `royaltyRecipient_` argument
This may cause a loss in royalty fees when `royaltyRecipient` is set to zero address
