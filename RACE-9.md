**Note**: All 8 questions in this RACE are based on the below contracts. This is the same contracts you will see for all the 8 questions in this RACE. The question is below the shown contracts.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.7;

import "@openzeppelin/contracts/access/Ownable.sol";

// Assume the Proxy contract was deployed and no further transactions were made afterwards.

contract Mastercopy is Ownable {
   int256 public counter = 0;

   function increase() public onlyOwner returns (int256) {
       return ++counter;
   }

   function decrease() public onlyOwner returns (int256) {
       return --counter;
   }

}

contract Proxy is Ownable {
   mapping(bytes4 => address) public implementations;

   constructor() {
       Mastercopy mastercopy = new Mastercopy();
       implementations[bytes4(keccak256(bytes("counter()")))] = address(mastercopy);
       implementations[Mastercopy.increase.selector] = address(mastercopy);
       implementations[Mastercopy.increase.selector] = address(mastercopy);
   }

   fallback() external payable {
       address implementation = implementations[msg.sig];

       assembly {
           // Copied without changes to the logic from OpenZeppelin's Proxy contract.
           calldatacopy(0, 0, calldatasize())
           let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)
           returndatacopy(0, 0, returndatasize())
           switch result
               case 0 {
                   revert(0, returndatasize())
               }
               default {
                   return(0, returndatasize())
               }
       }
   }

   function setImplementationForSelector(bytes4 signature, address implementation) external onlyOwner {
       implementations[signature] = implementation;
   }
}
```

---

**[Q1] The function signature is the first 4 bytes of the keccak hash which**

(A): Includes the function name  
(B): Includes a comma separated list of parameter types  
(C): Includes a comma separated list of return value types  
(D): Is generated only for public and external functions  

<details><summary><b>[Answers]</b></summary><b>
A, B, D
</b></details>

---

**[Q2] The Proxy contract is most similar to a**

(A): UUPS Proxy  
(B): Beacon Proxy  
(C): Transparent Proxy  
(D): Metamorphic Proxy  

<details><summary><b>[Answers]</b></summary><b>
C
</b></details>

---

**[Q3] Gas will be saved with the following changes**

(A): Skipping initialization of counter variable  
(B): Making `increase()` function external to avoid copying from calldata to memory  
(C): Packing multiple implementation addresses into the same storage slot  
(D): Moving the calculation of the `counter()` function's signature hash to a constant  

<details><summary><b>[Answers]</b></summary><b>
A
</b></details>

---

**[Q4] Calling the `increase()` function on the Proxy contract**

(A): Will revert since the Proxy contract has no `increase()` function  
(B): Will revert for any other caller than the one that deployed the Proxy  
(C): Increases the integer value in the Proxy's storage slot located at index 1  
(D): Delegate-calls to the zero-address  

<details><summary><b>[Answers]</b></summary><b>
B, C
</b></details>

---

**[Q5] Calling the `decrease()` function on the Proxy contract**

(A): Will revert because it was not correctly registered on the proxy  
(B): Will succeed and return the value of counter after it was decreased  
(C): Will succeed and return the value of counter before it was decreased  
(D): Will succeed and return nothing  

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q6] Due to a storage clash between the Proxy and the Mastercopy contracts**

(A): Proxy's `implementations` would be overwritten by 0 during initialization of the Mastercopy  
(B): Proxy's `implementations` would be overwritten when the counter variable changes  
(C): Proxy's `implementations` variable's storage slot being overwritten causes a DoS  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q7] The Proxy contract**

(A): Won't be able to receive any ether when `calldatasize` is 0 due to a missing `receive()`  
(B): Will be the owner of the Mastercopy contract  
(C): Has a storage clash in slot 0 which will cause issues with the current Mastercopy  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>

---

**[Q8] The `fallback()` function's assembly block**

(A): Can be marked as "memory-safe" for gas optimizations  
(B): Has the result of the delegate-call overwrite the the call parameters in memory  
(C): Interferes with the Slot-Hash calculation for the implementations-mapping by overwriting the scratch space  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>

---
