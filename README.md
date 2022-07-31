# Solidity Attack Vectors

Collection of Solidity Attack Vectors gathered from various sources. 
Make a pull request in order to contribute.

## 1 - Default Function Visibility
In solc < 0.5.0 functions are public by default if a function visibility type is not specified. If a developer neglected to specify the visibility and a malevolent user is able to perform illegal or unintentional state changes, this might result in a vulnerability.

Explicit function visibility specifiers are necessary for solc >= 0.5.0.


## 2 - Integer Overflow/Underflow
When an arithmetic operation exceeds or falls short of the type's maximum or minimum size, an overflow or underflow occurs.

For example - `uint8 -> (2^8)-1` `uint256 -> (2^256)-1`

When you go over the maximum amount (overflow) or below the minimum value (underflow), errors called overflow and underflow might occur. You return to zero when you go beyond the maximum value, and you return to the maximum value when you go below the minimum value.

All arithmetic operations now have default overflow/underflow checks thanks to solc v0.8.0.

## 3 - Solidity Versions 
**Outdated Versions:**
Solidity bug patches and updated security checks are not available if you are using extremely old versions of the language.
Check the list of bugs from past versions [here](https://docs.soliditylang.org/en/latest/bugs.html).

**Newer Versions:**
Using the most recent versions might expose contracts to unforeseen compiler problems.
Here is a list of recommended versions to use by [silther](https://github.com/crytic/slither/wiki/Detector-Documentation#recommendation-99).

```solidity
//Floating pragma
pragma solidity ^0.8.4;

//Fixed pragma
pragma solidity 0.8.4;

```

## 4 - Floating Pragma
A floating pragma makes it possible for contracts to unintentionally be deployed using a faulty or out-of-date compiler version, which might result in defects and compromise the security of your smart contract. 

Selecting and sticking with a single compiler version is regarded as best practise
When a contract is meant to be used by other developers, as is the case with contracts in a library or EthPM package, pragma declarations may be permitted to float.

## 5 - Unchecked Call Return Value
If a low-level call's return value is not verified, execution may continue even if the function call throws an error. This may result in unexpected behaviour and damage the logic of the program.
Make sure to manage the potential that the call may fail by validating the return value if you want to employ low-level call methods.

**Code Example:**
```solidity
//Not checking return value
function unchecked(address callee) public {
    callee.call();
}

//Checking return value
function checked(address callee) public {
    require(callee.call());
}

```

## 6 - 
