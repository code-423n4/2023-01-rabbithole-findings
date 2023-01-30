# \[NC\] Quest.constructor: ```endTime_ > block.timestamp``` check is redundant
Currently

```solidity
// Quest constructor
    if (endTime_ <= block.timestamp) revert EndTimeInPast();
    if (startTime_ <= block.timestamp) revert StartTimeInPast();
    if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
```
This means thats next checks are done:
1. ```endTime_ > block.timestamp```
2. ```startTime_ > block.timestampp```
3. ```endTime_ > startTime_```

From 2 and 3: ```endTime_ > startTime_ > block.timestampp```
Then 1 can be omitted. Line ```if (endTime_ <= block.timestamp) revert EndTimeInPast();``` should be removed, as well as ```EndTimeInPast()``` error declaration.}

# \[Non-critical\] ```Quest.start```, ```Quest.pause```, ```Quest.unPause``` should emit events
Given that this calls are significant for multiple core contract functionalities, emiting events when this function are called informing the quest start and its pause/unpause would be advisable.

# \[Non-critical\] ```Quest.pause```, ```Quest.unPause```, ```Quest.start``` can be called at any time
```Quest.pause``` should check that ```isPaused = false```, while ```Quest.unPause``` should check that ```isPaused = true``` in order to execute a meaningful transaction.

```Quest.start``` should check that the quest has not already started.


# \[Low\] ```Erc20Quest.withdrawFee``` allows anyone to call this function
Modifier ```onlyAdminWithdrawAfterEnd``` does not check the admin/owner, meaning anyone can call ```withdrawFee``` function.

Mittigation steps: Add ```onlyOwner``` modifier or check ownership in ```onlyAdminWithdrawAfterEnd``` modifier.


# \[Non-critical\] ```RabbitHoleTickets.royaltyInfo(uint256,uint256)``` remove first parameter
```tokenId_``` is not used inside the function, then it can be removed
```diff
function royaltyInfo(
-   uint256 tokenId_,
    uint256 salePrice_
) external view override returns (address receiver, uint256 royaltyAmount) {
    uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
    return (royaltyRecipient, royaltyPayment);
}
```

# \[Low\] ```RabbitHoleTickets``` intitalizer and setters should check non zero address assignation
```diff
    function initialize(
        address ticketRenderer_,
        address royaltyRecipient_,
        address minterAddress_,
        uint royaltyFee_
    ) public initializer {
        __ERC1155_init('');
        __Ownable_init();
        __ERC1155Burnable_init();
+       require(ticketRenderer_ != address(0), ERROR_ADDRESS_ZERO_ASSIGNATION());
+       require(royaltyRecipient_ != address(0), ERROR_ADDRESS_ZERO_ASSIGNATION());
+       require(minterAddress_ != address(0), ERROR_ADDRESS_ZERO_ASSIGNATION());
        royaltyRecipient = royaltyRecipient_;
        minterAddress = minterAddress_;
        royaltyFee = royaltyFee_;
        TicketRendererContract = TicketRenderer(ticketRenderer_);
    }
//...

    function setTicketRenderer(address ticketRenderer_) public onlyOwner {
+       require(ticketRenderer_ != address(0), ERROR_ADDRESS_ZERO_ASSIGNATION());
        TicketRendererContract = TicketRenderer(ticketRenderer_);
    }
//...

    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
+       require(royaltyRecipient_ != address(0), ERROR_ADDRESS_ZERO_ASSIGNATION());
        royaltyRecipient = royaltyRecipient_;
    }
//...
    function setMinterAddress(address minterAddress_) public onlyOwner {
+       require(minterAddress_ != address(0), ERROR_ADDRESS_ZERO_ASSIGNATION());
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```

# \[Non-critical\] ```RabbitHoleTickets``` setters should check if value are really changed
```diff
    function setTicketRenderer(address ticketRenderer_) public onlyOwner {
        require(ticketRenderer_ != address(TicketRendererContract));
        TicketRendererContract = TicketRenderer(ticketRenderer_);
    }
    //...

    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
+       require(royaltyRecipient != royaltyRecipient_);
        royaltyRecipient = royaltyRecipient_;
    }

    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
+       require(royaltyFee != royaltyFee_);
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
//...

    function setMinterAddress(address minterAddress_) public onlyOwner {
+       require(minterAddress_ != minterAddress);
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```

# \[Low\] Lack of event emission when ```setTicketRenderer``` and ```setRoyaltyRecipient``` are called
These seem important methods given they modifed state variables, it would be advisable to emit and event indicating the new values.

