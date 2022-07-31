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

## 6 - Unprotected Ether Withdrawal
Malicious parties may remove all or part of the Ether from the contract account due to missing or inadequate access constraints.

Put safeguards in place so that withdrawals may only be triggered by permitted parties or in accordance with the smart contract system's specifications.

## 7 - Unprotected call to 'selfdestruct'
Malicious actors may self-destruct contracts that feature a selfdestruct method if there aren't any or enough access controls.

If the self-destruct feature isn't absolutely necessary, consider about removing it. It is advised to construct a multisig scheme if there is a legitimate use-case such that many parties must consent to the self-destruct operation.

## 8 - Default State Variable Visibility
It is simpler to spot false assumptions about who may access the variable when the visibility is explicitly labelled.
Variables can be designated as `internal`, `private`, or `public`. All state variables should have visibility defined explicitly.

## 9 - Uninitialized Storage Pointer
The EVM can store data as `storage`, `memory`, or `calldata`.
Uninitialized local storage variables may point to unintended contract storage locations, creating security holes. 

This problem has been systematically fixed as of compiler version 0.5.0 and above since contracts with uninitialized storage references no longer compile.

## 10 - Assert Violation
Invariants are asserted using the Solidity `assert()` function. A failed assert statement should never be encountered in properly working code. A reachable assertion may refer to one of the following:
The `assert` statement is improperly used, for example, to check inputs. A defect in the contract causes it to enter an invalid state.
Examine if the `assert()` condition properly checks for an invariant. If not, substitute a require() statement for the assert() one.
Fix the underlying bug(s) that allow the assertion to be broken if the exception is in fact brought on by the code acting in an unanticipated way.

## 11 - Use of Deprecated Solidity Functions
In Solidity, a number of operations and functions have been deprecated. They result up in bad code quality when used. Deprecated operators and functions may have negative side effects and compilation issues with new major versions of the Solidity compiler.

Solidity provides alternatives to the deprecated constructions.
| Deprecated              | Alternatives              |
| :---------------------- | ------------------------: |
| `suicide(address)`      |	`selfdestruct(address)`   |
| `block.blockhash(uint)` |	`blockhash(uint)`         |
| `sha3(...)`	            | `keccak256(...)`          |
| `callcode(...)`	        | `delegatecall(...)`       |
| `throw`	                | `revert()`                |
| `msg.gas`	              | `gasleft`                 |
| `constant`              | `view`                    |
| `var`	                  | `corresponding type name` |

## 12 - Delegatecall to Untrusted Callee
Malicious contracts can be executed in the context of the caller's state by calling `delegatecall()` or `callcode()` to an address within the user's control. For such calls, make sure the destination addresses are reliable.

Be careful while using `delegatecall` and never call into suspicious contracts. Make care to cross-reference the destination address with a whitelist of trustworthy contracts if it was obtained via user input.

## 13 - Transaction Order Dependence AKA Front-Running
On the Ethereum network, transactions are processed in blocks, with new blocks being confirmed every 17 seconds. The transactions that have paid a high enough gas price to be included are those that the miners choose to include in a block after reviewing the transactions they have received. Additionally, each node receives and processes the transactions that are submitted to the Ethereum network. This means that someone running an Ethereum node may predict which transactions will take place before they are confirmed. When a piece of code depends on how transactions are sent to it, there is a race situation vulnerability.

This technique, for instance, may be used to front-run the traditional ERC20 `approve()` change.

## 14 - Authorization through tx.origin
The address of the account that sent the transaction is returned by the global variable `tx.origin` in the Solidity .If an authorised account calls into a malicious contract, using the variable for authorization might render a contract susceptible. Since `tx.origin` provides the transaction's original sender, in this instance the approved account, it is possible to call the susceptible contract that successfully passes the authorisation check.

`tx.origin` should not be used for authorization. Use `msg.sender` instead.

## 15 - Block values as a proxy for time
Developers frequently try to utilise the `block.timestamp` function to start time-related activities. Due to Ethereum's decentralised nature, nodes may only partially synchronise time. Furthermore, malicious miners may alter the timestamp of their blocks if doing so would benefit them. However, miners are not permitted to set a timestamp that is less recent than the preceding one (doing so would result in the block being rejected) nor can they establish a timestamp that is too distant in the future. Developers cannot rely on the accuracy of the given timestamp in light of the foregoing.

## 16 - Signature Malleability
Although it is frequently assumed that each signature is unique when a cryptographic signature scheme is implemented in Ethereum contracts, signatures can be changed without having access to the private key and still be considered legitimate. One of the so-called "precompiled" contracts defined by the EVM standard is `ecrecover`, which implements elliptic curve public key recovery. The three values v, r, and s can be slightly altered by a malicious user to produce different valid signatures. If the signature is a component of the hash of the signed message, a system that verifies signatures at the contract level may be vulnerable to attacks. A malevolent user might produce legitimate signatures to replay previously signed communications.

## 17 - Shadowing State Variables
When inheritance is implemented, Solidity permits ambiguous naming of state variables. Contract A, which likewise has a state variable x declared, might inherit contract B, which contains a variable x. As a result, there would be two distinct copies of x, one of which could be accessed through contract A and the other through contract B. This circumstance might go unreported in more intricate contract systems, which could result in security vulnerabilities.

**Code Example:**
```solidity
contract A {
    uint public x = 1;
}

contract B is A {
    uint public x = 2;
}

```

## 18 - Bad Randomness
In a wide range of applications, having the ability to create random numbers is quite useful. Gambling DApps, which choose the winner using a pseudo-random number generator, are one clear example. It is difficult to build a reliable adequate source of randomness for Ethereum, nevertheless. A miner may choose to submit any timestamp within a few seconds and still have his block accepted by others, making the usage of `block.timestamp`, for instance, unsafe. Since the miner controls `blockhash`, `block.difficulty`, and other variables, their use is equally risky. If the stakes are high, the miner can quickly mine a large number of blocks using rental gear, choose the block with the requisite block hash, and dump all other blocks.

## 19 - Signature Replay Attacks
To increase usability and reduce gas costs, signature verification may occasionally be required in smart contracts. However, while putting signature verification into practise, care must be taken. The contract should only permit the processing of fresh hashes in order to defend against Signature Replay Attacks. This stops malevolent users from repeatedly repeating the signature of another user.

## 20 - Requirement Violation
The `require()` function is used to verify inputs, contract state variables, return values from external contract calls, and other criteria. Inputs can be given by callers or returned by callees to validate external calls. If a callee's return value displays an input violation, one of two things has probably gone wrong:
1. A bug exists in the contract that provided the external input.
2. The condition used to express the requirement is too strong.

## 21 - Write to Arbitrary Storage Location
The data of a smart contract is consistently saved at some storage place (i.e., a key or address) on the EVM level, such as the owner of the contract. Only authorised users or contract accounts are permitted to write to sensitive storage locations, and this responsibility rests with the contract. The permission checks may be readily bypassed if an attacker has access to write to any arbitrary storage places in a contract. Through the overwriting of a field that contains the address of the contract owner, for example, an attacker may use this to corrupt the storage.

## 22 - Incorrect Inheritance Order
Multiple inheritance is supported by Solidity, allowing for the inheritance of multiple contracts by a single contract. Multiple inheritance poses an ambiguity known as the Diamond Problem: which base contract should be called in the child contract if two or more specify the identical function? Reverse C3 Linearization, which establishes a precedence amongst basic contracts, is the method used by Solidity to resolve this problem.

In this approach, the sequence of inheritance matters since base contracts have distinct priorities. Unexpected behaviour might result from disregarding inheritance order.

## 23 - Insufficient Gas Griefing
Contracts that take data and use it in a sub-call on another contract are susceptible to griefing attacks with insufficient gas. If the sub-call is unsuccessful, either the entire transaction is revoked, or execution is continued. By utilising just enough gas to carry out the transaction but not enough for the sub-call to succeed, the user that conducts the transaction, known as the "forwarder," can effectively filter transactions in the event of a relayer contract.
There are two options to prevent insufficient gas griefing:
1. Only allow trusted users to relay transactions.
2. Require that the forwarder provides enough gas.

## 24 - Unencrypted Private Data On-Chain
Contrary to popular belief, `private` type variables can be read. Attackers can look at contract transactions or storage locations even if your contract is not public to identify values kept in the state of the contract. It's crucial that no unencrypted private information be kept in the contract code or state for this reason.
Any private data should either be stored off-chain, or carefully encrypted.

## 25 - Message call with hardcoded gas amount
Using the `transfer()` and `send()` procedures, 2300 gas are sent in fixed amounts. To protect against reentrancy attacks, it has historically been advised to employ these functions for value transfers. However, during hard forks, the gas cost of EVM instructions may fluctuate dramatically, which might lead to the failure of already-implemented contract systems that rely on constant gas cost assumptions. An illustration. Due to an increase in the cost of the SLOAD instruction, EIP 1884 broke a number of existing smart contracts.
Avoid the use of transfer() and send() and do not otherwise specify a fixed amount of gas when performing calls. Use .call.value(...)("") instead.

## 26 - Arbitrary Jump with Function Type Variable
Function types are supported by Solidity. In other words, a reference to a function with a matching signature can be given to a variable of function type. It is possible to call the function stored to such a variable exactly like any other function.

The issue emerges when a user has the power to alter the function type variable at will, causing arbitrary code instructions to be executed. Since Solidity doesn't allow pointer arithmetic, it is impossible to assign an arbitrary value to such a variable. However, in the worst case scenario, an attacker might point a function type variable to any code instruction, breaking necessary validations and necessary state changes, if the developer utilises assembly instructions, such as `mstore` or the assign operator.

## 27 - DoS With Block Gas Limit
A certain quantity of gas is always required for the execution of smart contract deployments and function calls inside them, depending on the amount of computation involved. The total number of transactions contained in a block cannot go over the block gas limit set by the Ethereum network.

When the cost of running a function exceeds the block gas limit, programming patterns that are safe in centralised apps might cause Denial of Service problems in smart contracts. A Denial of Service situation like this might result from changing an array whose size is unknown and which grows over time.

## 28 - Unexpected Ether balance
When contracts rigorously presume a particular Ether balance, they may operate incorrectly. It is always possible to use selfdestruct, mining to the account, or forcefully deliver ether to a contract (without using its fallback mechanism). In the worst instance, this can result in DOS circumstances that make the contract useless.

## 29 - Hash Collisions With Multiple Variable Length Arguments
In some cases, using `abi.encodePacked()` with several parameters of different lengths might result in a hash collision. Because `abi.encodePacked()` packs all items in order whether or not they are part of an array, you may transfer elements across arrays and it will still return the same encoding as long as all of the components are in the same order. An attacker might take advantage of this in a signature verification scenario by changing the order of items in a prior function call to successfully evade permission.
