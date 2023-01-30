Hi team, I think I have found a few low risk bugs, and have included a short write-up below. 
PS: this is my first time submitting a bug bounty so any feedback will be greatly appreciated.

**[L-1] ERC20 Quest: withdrawFee() function should only be able to be called once instead of multiple times**
> Issue: The withdrawFee() function can be called multiple times by admin after a quest ends, resulting in more than the protocolFee being paid. This will help to prevent potential abuse or accidental calling the function more than once.

> Suggested Fix: include a boolean check e.g. feeWithdrawn = True so that the withdrawFee() function can only be called once

    /// @notice add bool
    bool public feeWithdrawn;

    /// @notice Starts the Quest
    /// @dev Only the owner of the Quest can call this function
    function start() public virtual onlyOwner {
        isPaused   = false;
        hasStarted = true;
        feeWithdrawn = false;
    }

    /// @notice Sends the protocol fee to the protocolFeeRecipient
    /// @dev Only callable when the quest is ended
    function withdrawFee() public onlyAdminWithdrawAfterEnd {        
        if (feeWithdrawn) revert FeeAlreadyWithdrawn();
        IERC20(rewardToken).safeTransfer(protocolFeeRecipient, protocolFee());
        feeWithdrawn = true;
    }

> Link to Github Reference: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L100-L104


**[L-2] ERC1155 Quest: withdrawRemainingTokens should factor in total number of receipts minted before withdrawal**
>  Issue: There may be users with unredeemed receipts who will not be able to claim if all the remaining tokens are withdrawn to the owner after a quest has ended. Ideally, the process should be the same as ERC20 where this is taken into account.

> Suggested Fix: Include the Quest Factory contract in the ERC1155 contract as well (e.g. through the constructor similar to receiptContractAddress_)

> In withdrawRemainingTokens() function: 

		uint unclaimedTokens       = (receiptRedeemers() - redeemedTokens);
 		uint256 nonClaimableTokens = IERC1155(rewardToken).balanceOf(address(this)) - unclaimedTokens;
		IERC1155(rewardToken).safeTransferFrom(address(this), to_, nonClaimableTokens, nonClaimableTokens, '0x00');

		/// @notice Call the QuestFactory contract to get the amount of receipts that have been minted
		/// @return The amount of receipts that have been minted for the given quest
	    	function receiptRedeemers() public view returns (uint256) {
        		return questFactoryContract.getNumberMinted(questId);
		 }
> Link to Github Reference: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L52-L63


**[L-3] totalParticipants for any quest should be checked that it does not exceed total receipts minted**

> Issue: While the totalParticipants variable is set upon creation of a quest to determine the max total rewards, there are no checks to ensure that a quest’s total receipts minted does not exceed its total participants number. This may result in scenarios where users are unable to claim their quest reward even if they have successfully completed the action.

> Suggested Fix: Check the numberMinted of a quest against it’s totalParticipants in the mint() function in RabbitHoleReceipt contract

             (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
             IQuest questContract = IQuest(questAddress);
             uint numberMinted = questContract.numberMinted;
             uint totalParticipants = questContract.totalParticipants;
             if (numberMinted >= totalParticipants) revert TotalParticipantsReached();

> Link to Github Reference: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L95-L104
