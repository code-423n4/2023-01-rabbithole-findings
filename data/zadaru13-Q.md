<h1>pragma solidity ^ should not be used</h1>

Currently: pragma solidity ^0.8.15 was used across all files

Solution: Should not use ^ due as some functions may be deprecate and will not be able to function as it is

[S-01] MAKE THE TEST CONTEXT WITH SOLIDITY
It’s crucial to write tests with possibly 100% coverage for smart contract systems.
It is recommended to write appropriate tests for all possible code streams and especially for extreme cases.
But the other important point is the test context.
Tests written with Solidity are safer, so it is recommended to focus on tests with Foundry.

[S-03] GENERATE PERFECT CODE HEADERS EVERY TIME
I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/


[N-11] LONG LINES ARE NOT SUITABLE FOR THE SOLIDITY STYLE GUIDE
It is generally recommended that lines in the source code should not exceed 80-120 characters. Today’s screens are much larger, so in some cases it makes sense to expand that. The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.

See why-is-80-characters-the-standard-limit-for-code-width.

[N-05] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last