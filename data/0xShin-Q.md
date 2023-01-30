# Impact
The naming of the grantDefaultAdminAndCreateQuestRole function should begin with an underscore.

## Proof of Concept


```
    function grantDefaultAdminAndCreateQuestRole(address account_) internal {
        _grantRole(DEFAULT_ADMIN_ROLE, account_);
        _grantRole(CREATE_QUEST_ROLE, account_);
    }
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152


## Tools Used
Manual review

## Recommended Mitigation Steps
To improve code clarity and function visibility, it is recommended to rename the internal functions to follow best practices, by starting the name with an underscore.
