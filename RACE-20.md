**Note**: The first four questions are based on the below library. The same library will appear for all the first four questions. The question is below the shown library.
```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.0;

/// @notice Efficient library for creating string representations of integers.
/// @author Solmate (https://github.com/Rari-Capital/solmate/blob/v7/src/utils/LibString.sol)
library LibString {
    function toString(uint256 n) internal pure returns (string memory str) {
        if (n == 0) return "0"; // Otherwise it'd output an empty string for 0.

        assembly {
            let k := 78 // Start with the max length a uint256 string could be.

            // We'll store our string at the first chunk of free memory.
            str := mload(0x40)

            // The length of our string will start off at the max of 78.
            mstore(str, k)

            // Update the free memory pointer to prevent overriding our string.
            // Add 128 to the str pointer instead of 78 because we want to maintain
            // the Solidity convention of keeping the free memory pointer word aligned.
            mstore(0x40, add(str, 128))

            // We'll populate the string from right to left.
            // prettier-ignore
            for {} n {} {
                // The ASCII digit offset for '0' is 48.
                let char := add(48, mod(n, 10))

                // Write the current character into str.
                mstore(add(str, k), char)

                k := sub(k, 1)
                n := div(n, 10)
            }

            // Shift the pointer to the start of the string.
            str := add(str, k)

            // Set the length of the string to the correct value.
            mstore(str, sub(78, k))
        }
    }
}
```
---
**[Q1] Select all true statements:** \
(A): The inline assembly block is `memory-safe` \
(B): The memory after `toString(...)` call is always 32-byte aligned \
(C): Instead of allocating memory from 0x40, the function can allocate from 0x0 to save gas (memory expansion cost) and still be correct \
(D): None of the above 

**[Answers]: A**

---
**[Q2] Select all true statements about the expression `mstore(0x40, add(str, 128))`** \
(A): The expression allocated more memory than required. The value 128 can be replaced by 96. \
(B): The expression allocates less memory than required. The value 128 can be replaced by 160. \
(C): The expression is redundant and can be removed to save gas \
(D): The expression is not `memory-safe` assembly in this context 

**[Answers]: B**

---
**[Q3] Select all true statements:** \
(A): The expression `mstore(str, k)` at the beginning can be removed to save gas \
(B): The expression `mstore(add(str, k), char)` can be replaced by an equivalent `mstore8(...)` to simplify the code \
(C): The final expression `mstore(str, sub(78, k))` can be removed to save gas \
(D): The function does not return the correct output for `n = 2**256 - 1`

**[Answers]: A, B**

---
**[Q4] Select all true statements:** \
(A): The function correctly cleans all necessary memory regions \
(B): Solidity will correctly be able to handle the string returned by the function \
(C): The last bits of memory in the string may be dirty \
(D): None of the above

**[Answers]: B, C**

---
**Note**: The last four questions are based on the below abstract contract. The same abstract contract will appear for all the last four questions. The question is below the shown abstract contract.
```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

abstract contract Proxy is ReentrancyGuard {
    function _delegate(address implementation) nonReentrant internal virtual {
        assembly ("memory-safe") {
            // Copy msg.data. We take full control of memory in this inline assembly
            // block because it will not return to Solidity code. We overwrite the
            // Solidity scratch pad at memory position 0.
            calldatacopy(0, 0, calldatasize())

            // Call the implementation.
            // out and outsize are 0 because we don't know the size yet.
            let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)

            // Copy the returned data.
            returndatacopy(0, 0, returndatasize())

            switch result
            // delegatecall returns 0 on error.
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }
}
```
**[Q5] Select all true statements:** \
(A): The re-entrancy lock is always unnecessary as it’s never possible to re-enter the contract \
(B): Calls to `_delegate` are correctly protected for re-entrancy \
(C): The re-entrancy lock is correctly unlocked in some cases \
(D): The re-entrancy lock is correctly unlocked in all cases

**[Answers]: C**

---
**[Q6] Select all true statements:** \
(A): The assembly block is correctly marked as `memory-safe` \
(B): The assembly block will always violate the memory requirements needed for `memory-safe` blocks \
(C): In some cases, the assembly block will not violate the requirement needed for `memory-safe` blocks \
(D): None of the above

**[Answers]: C**

---
**[Q7] Select all true statements:** \
(A): The expression `calldatacopy(0, 0, calldatasize())` violates `memory-safe` assembly annotation \
(B): The expression `returndatacopy(0, 0, returndatasize())` violates `memory-safe` assembly annotation \
(C): The expression `delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)` violates `memory-safe` assembly annotation \
(D): The expression `return(0, returndatasize())` violates `memory-safe` assembly annotation \
(E): The expression `revert(0, returndatasize())` violates `memory-safe` assembly annotation \
(F): None of the above

**[Answers]: A, B**

---
**[Q8] Select all true statements:** \
(A): `delegatecall` can never re-enter as the state is shared \
(B): `delegatecall` proxies without proper access controls may be prone to `selfdestruct` \
(C): Proxies are typically used to save deploy-time gas costs \
(D): Proxies can be used to prevent contract size limit issues

**[Answers]: B, C, D**

---
