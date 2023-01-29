### No same value input control
```
QuestFactory.sol

    function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
        if (canCreateQuest_) {
            _grantRole(CREATE_QUEST_ROLE, account_);
        } else {
            _revokeRole(CREATE_QUEST_ROLE, account_);
        }
    }

    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
        claimSignerAddress = claimSignerAddress_;
    }

   function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
        if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();
        protocolFeeRecipient = protocolFeeRecipient_;
    }

   function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
        rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
    }

   function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
        rewardAllowlist[rewardAddress_] = allowed_;
    }

RabbitHoleReceipt.sol
  function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
        ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
    }

    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
        royaltyRecipient = royaltyRecipient_;
    }

    function setQuestFactory(address questFactory_) public onlyOwner {
        QuestFactoryContract = IQuestFactory(questFactory_);
    }

    function setQuestFactory(address questFactory_) public onlyOwner {
        QuestFactoryContract = IQuestFactory(questFactory_);
    }

    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }

RabbitHoleTickets.sol
    function setTicketRenderer(address ticketRenderer_) public onlyOwner {
        TicketRendererContract = TicketRenderer(ticketRenderer_);
    }

    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
        royaltyRecipient = royaltyRecipient_;
    }

    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }


```