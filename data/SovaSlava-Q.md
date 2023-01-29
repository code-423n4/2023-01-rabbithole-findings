Modifier  onlyAdminWithdrawAfterEnd() in Quest.sol dont have check of msg.sender. Advice to rename onlyAdminWithdrawAfterEnd to withdrawAfterEnd
```
    modifier onlyAdminWithdrawAfterEnd() {
        if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
        _;
//  NO CHECKS MSG.SENDER == OWNER
    }
```
its not critical, because receiver of fee is set in constructor
