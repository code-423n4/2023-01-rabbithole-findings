**QuestFactory**
- L142 - Inside the function the inputs are used in a different order, therefore the most serious thing is to change the orders (canCreateQuest_, account_)


**RabbitHoleReceipt**
- L35/36 - Two variables are created in storage that begin with a capital letter (QuestFactoryContract and ReceiptRendererContract) this does not meet the standards, it should be written in lowercase, with camemel case.

- L59 - Within the modifier the validation msg.sender == minterAddress is performed, but in any case an error message should be displayed in case it reverts.

- L109 - It is not very intuitive in the function that the questId_ is passed first and then the claimingAddress_. The most common would be to pass the address parameter first.


**Quest**
- L7 - ECDSA is imported, but it is never used, so it should be removed.

- L63/97 - The project has a degree of centralization, since the owner of the contract can pause the contract and disable the claims, this does not have theft problems, but there are problems with blocking funds. Therefore, anyone who wants to use these funds has to trust that the project does not want to lock up the funds.


**RabbitHoleTickets**
- L48 - Within the modifier the validation msg.sender == minterAddress is performed, but in any case an error message should be displayed in case it reverts.

