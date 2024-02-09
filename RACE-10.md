**Note**: All 8 questions in this RACE are based on the below contract. This is the same contract you will see for all the 8 questions in this RACE. The question is below the shown contracts.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;
import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";

contract TestContract is Ownable {

    function Test1(uint n) external pure returns(uint) {
        return n + abi.decode(msg.data[msg.data.length-64:], (uint));
    }

    function Test2(uint n) public view returns(uint) {
        bytes memory fcall = abi.encodeCall(TestContract.Test1,(n));
        bytes memory xtr = abi.encodePacked(uint(4),uint(5));
        bytes memory all = bytes.concat(fcall,xtr);
        (bool success, bytes memory data) = address(this).staticcall(all);
        return abi.decode(data,(uint));
    }

    type Nonce is uint256;
    struct Book { Nonce nonce;}

    function NextBookNonce(Book memory x) public pure {
       x.nonce = Nonce.wrap(Nonce.unwrap(x.nonce) + 3);
    }

    function Test3(uint n) public pure returns (uint) {
      Book memory bookIndex;
      bookIndex.nonce = Nonce.wrap(7);
      for (uint i=0;i<n;i++) {
         NextBookNonce(bookIndex);
      }
      return Nonce.unwrap(bookIndex.nonce);
    }

    error ZeroAddress();
    error ZeroAmount();
    uint constant ZeroAddressFlag = 1;
    uint constant ZeroAmountFlag = 2;

    function process(address[] memory a, uint[] memory amount) public pure returns (uint){
        uint error;
        uint total;
        for (uint i=0;i<a.length;i++) {
            if (a[i] == address(0)) error |= ZeroAddressFlag;
            if (amount[i] == 0) error |= ZeroAmountFlag;
            total += amount[i];
        }
        if (error == ZeroAddressFlag) revert ZeroAddress();
        if (error == ZeroAmountFlag)  revert ZeroAmount();
        return total;
    }

    function Test4(uint n) public pure returns (uint) {
        address[] memory a = new address[](n+1);
        for (uint i=0;i<=n;i++) {
            a[i] = address(uint160(i));
        }
        uint[] memory amount = new uint[](n+1);
        for (uint i=0;i<=n;i++) {
            amount[i] = i;
        }
        return process(a,amount);
    }

    uint public totalMinted;
    uint constant maxMinted = 100;
    event minted(uint totalMinted,uint currentMint);

    modifier checkInvariants() {
        require(!paused, "Paused");
        _;
        invariantCheck();
        require(!paused, "Paused");
    }

    function invariantCheck() public {
        if (totalMinted > maxMinted) // this may never happen
            pause();
    }

    bool public paused;
    function pause() public {
        paused = true;
    }
    function unpause() public onlyOwner {
        paused = false;
    }

    function Test5( uint n) public checkInvariants(){
        totalMinted += n;
        emit minted(n,totalMinted);
    }
}
```

---

**[Q1] Which statements are true in `Test1()`?**

(A): The function does not use all supplied extra data  
(B): The function can revert due to an underflow  
(C): The function can revert due to an overflow  
(D): The function accesses memory which it should not

<details><summary><b>[Answers]</b></summary><b>
A, B, C
</b></details>

---

**[Q2] Which statements are true in `Test2()`?**

(A): Result of `encodePacked` is deterministic  
(B): `abi.decode` always succeeds  
(C): It calls function `Test1()` without any problem  
(D): It should use `uint256` instead of `uint`

<details><summary><b>[Answers]</b></summary><b>
A, C
</b></details>

---

**[Q3] Which statements are true in `NextBookNonce()`?**

(A): `wrap` and `unwrap` cost additional gas  
(B): It is safe to use `unchecked`  
(C): There is something suspicious about the increment value  
(D): It could have used `x.nonce++`

<details><summary><b>[Answers]</b></summary><b>
B, C
</b></details>

---

**[Q4] Which statements are true in `Test3()`?**

(A): `bookIndex.nonce` is incremented in the loop  
(B): `bookIndex.nonce` cannot be incremented because `NextBookNonce` is `pure`  
(C): `i++` can be made `unchecked`  
(D): `memory` can be changed to `storage` without any other changes

<details><summary><b>[Answers]</b></summary><b>
A, C
</b></details>

---

**[Q5] Which statements are true In `Test4()`?**

(A): The function always reverts with `ZeroAddress()`  
(B): The function always reverts with `ZeroAmount()`  
(C): The function never reverts with `ZeroAddress()`  
(D): The function never reverts with `ZeroAmount()`

<details><summary><b>[Answers]</b></summary><b>
C, D
</b></details>

---

**[Q6] Which statements are true in `Test5()`?**

(A): modifier `checkInvariants` will pause the contract if too much is minted  
(B): modifier `checkInvariants` will never pause the contract  
(C): modifier `checkInvariants` will always pause the contract  
(D): There are more efficient ways to handle the require

<details><summary><b>[Answers]</b></summary><b>
B, D
</b></details>

---

**[Q7] Which statements are true about the owner?**

(A): The owner is initialized  
(B): The owner is not initialized  
(C): The owner cannot be changed  
(D): The owner can be changed

<details><summary><b>[Answers]</b></summary><b>
A, D
</b></details>

---

**[Q8] Which statements are true In `Test5()` and related functions?**

(A): `pause` is unsafe  
(B): `unpause` is unsafe  
(C): The `emit` is done right  
(D): The `emit` is done wrong

<details><summary><b>[Answers]</b></summary><b>
A, D
</b></details>

---
