## Overview

The smart contracts are well separated. Across all of the contracts the comments are descriptive which gives a clear idea of what the functions intend to do. This report is regarding the non-critical issues of the protocol.

## [L-01] LOSS OF PRECISION DUE TO ROUNDING

Inside `Erc20Quest.sol` :
      
      function maxProtocolReward() public view returns (uint256) {
        return (maxTotalRewards() * questFee) / 10_000;
      }

     function protocolFee() public view returns (uint256) {
           return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
    }

## [L-02] WRONG COMPARISON

`Erc20Quest.sol`

    /// @dev Requires that the balance of the rewards in the contract is greater than or equal to the maximum amount of rewards that can be claimed by all users and the protocol

      function start() public override {
        if (IERC20(rewardToken).balanceOf(address(this)) < maxTotalRewards() + maxProtocolReward())
            revert TotalAmountExceedsBalance();

As we can see from the comment, the balance of the contract should be greater than or equal to the maximum amount rewards. However, only the `<` sign is used and even if the amounts are equal the function will revert.

**Recommendation**

Add the `=` sign.


## [N-01] PREFER BATTLE-TESTED CODE OVER REIMPLEMENTING COMMON PATTERNS

Inside `Quest.sol` the functions `pause` and `unpause` can be replaced by implementing the OpenZeppelin Pausable smart contract, since it is well-tested and optimized.

## [N-02] UNUSED IMPORTS

Inside `Quest.sol` the imported OZ ECDSA library is unused. Consider removing it.

## [N-03] LOCK *PRAGMAS* TO SPECIFIC COMPILER VERSION

All of the contracts use floating pragmas `^0.8.15;`. As a best practice, it is always better to lock the pragma to a specific version.

## [N-04] OPEN TODO

Inside `IQuest.sol` interface there is an open TODO:
`// TODO clean this whole thing up`

**Recommendation**

Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.

## [N-05] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

        settings: {
          optimizer: {
            enabled: true,
            runs: 5000,
          },

Protocol has enabled optional compiler optimizations in Solidity.

There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised.

High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe.

It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

**Recommendation**

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [N-06] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## [N-07]MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

The `initialize()` function is marked as public in three of the contracts: `QuestFactory.sol`, `RabbitHoleReceipt.sol` and `RabbitHoleTickets.sol`.

If someone wants to extend via inheritance, it might make more sense that the overridden `initialize(...)` function calls the `internal {...}_init` function, not the parent public `initialize(...)` function.

External instead of public would give more sense of the `initialize(...)` functions to behave like a constructor (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

It might cost a bit less gas to use external over public.

It is possible to override a function from external to public (= “opening it up”) ✅ but it is not possible to override a function from public to external (= “narrow it down”). ❌

For the above reasons, you can change `initialize(...)` to external:

https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

## [N-08] INSUFFICIENT COVERAGE

The test coverage is relatively low and could be increased.

------------------------|----------|----------|----------|----------|----------------|
File                    |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------------------|----------|----------|----------|----------|----------------|
 contracts/             |    81.62 |    59.23 |    74.03 |    81.46 |                |
  Erc1155Quest.sol      |      100 |       75 |      100 |    85.71 |             35 |
  Erc20Quest.sol        |      100 |    66.67 |      100 |    94.12 |             60 |
  Quest.sol             |    90.91 |    81.58 |    73.33 |    90.48 |123,130,141,146 |
  QuestFactory.sol      |    94.29 |     72.5 |    85.71 |    94.23 |    146,173,200 |
  RabbitHoleReceipt.sol |    54.29 |    26.92 |    46.67 |    61.54 |... 184,185,193 |
  RabbitHoleTickets.sol |    58.33 |    18.75 |       50 |    59.09 |... 113,114,122 |
  ReceiptRenderer.sol   |      100 |      100 |      100 |      100 |                |
  TicketRenderer.sol    |      100 |      100 |      100 |      100 |                |
 contracts/interfaces/  |      100 |      100 |      100 |      100 |                |
  IQuest.sol            |      100 |      100 |      100 |      100 |                |
  IQuestFactory.sol     |      100 |      100 |      100 |      100 |                |
 contracts/test/        |       25 |        0 |       50 |    28.57 |                |
  SampleERC20.sol       |      100 |      100 |      100 |      100 |                |
  SampleErc1155.sol     |      100 |      100 |      100 |      100 |                |
  TestERC20.sol         |        0 |        0 |        0 |        0 |  8,14,15,16,18 |
------------------------|----------|----------|----------|----------|----------------|
All files               |    78.47 |    57.46 |    72.84 |    79.72 |                |
------------------------|----------|----------|----------|----------|----------------|