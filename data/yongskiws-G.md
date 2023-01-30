### [G-1] Use function instead of modifiers

EG
``` solidity

function onlyAdminWithdrawAfterEnd() private view {
      if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
}

function withdrawRemainingTokens(address to_) public virtual onlyOwner {
     onlyAdminWithdrawAfterEnd ();
}

```

replace modifier with private view function
``` solidity
quest\Quest.sol
76:     modifier onlyAdminWithdrawAfterEnd() {
77:         if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
78:         _;
79:     }
80: 
81:     /// @notice Checks if the Quest has started at the function level
82:     modifier onlyStarted() {
83:         if (!hasStarted) revert NotStarted();
84:         _;
85:     }
86: 
87:     /// @notice Checks if quest has started both at the function level and at the start time
88:     modifier onlyQuestActive() {
89:         if (!hasStarted) revert NotStarted();
90:         if (block.timestamp < startTime) revert ClaimWindowNotStarted();
91:         _;
92:     }


File: c:\Users\pc\Desktop\quest\RabbitHoleReceipt.sol
58:     modifier onlyMinter() {
59:         msg.sender == minterAddress;
60:         _;
61:     }


File: c:\Users\pc\Desktop\quest\RabbitHoleTickets.sol
47:     modifier onlyMinter() {
48:         msg.sender == minterAddress;
49:         _;
50:     }
```