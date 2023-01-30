# Use the most Recent Version of Solidity

## Context

All contracts

### Description

It is best practice to use the latest version of Solidity in order to take advantage of the latest security fixes and upgrades. Solidity version information can be found here https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### Recommendation

Use the latest version of solidity, which is 0.8.17


# Order of Functions Should Follow the Solidity Style Guide

## Context

All contracts

### Description

Per the solidity style guide - https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

constructor

receive function (if exists)

fallback function (if exists)

external

public

internal

private

Within a grouping, place the view and pure functions last.