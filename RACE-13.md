**Note**: All 8 questions in this RACE are based on the below contract. This is the same contract you will see for all the 8 questions in this RACE. The question is below the shown contract.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.17;

/// CallbackERC20 is based on Solmate's ERC20.
/// It adds the extra feature that address can register callbacks,
/// which are called when that address is a recipient or sender
/// of a transfer.
/// Addresses can also revoke callbacks.
contract CallbackERC20 {
   event Transfer(address indexed from, address indexed to, uint256 amount);

   event Approval(address indexed owner, address indexed spender, uint256 amount);

   /*//////////////////////////////////////////////////////////////
                           METADATA STORAGE
   //////////////////////////////////////////////////////////////*/

   // Optimized version vs string
   function name() external pure returns (string memory) {
       // Returns "Callback"
       assembly {
           mstore(0, 0x20)
           mstore(0x28, 0x0843616c6c6261636b)
           return(0, 0x60)
       }
   }

   // Optimized version vs string
   function symbol() external pure returns (string memory) {
       // Returns "CERC"
       assembly {
           mstore(0, 0x20)
           mstore(0x24, 0x0443455243)
           return(0, 0x60)
       }
   }

   uint8 public constant decimals = 18;

   /*//////////////////////////////////////////////////////////////
                             ERC20 STORAGE
   //////////////////////////////////////////////////////////////*/

   uint256 public totalSupply;

   mapping(address => uint256) public balanceOf;

   mapping(address => mapping(address => uint256)) public allowance;

   /*//////////////////////////////////////////////////////////////
                              CALLBACK
   //////////////////////////////////////////////////////////////*/

   mapping(address => function (address, address, uint) external) public callbacks;

   /*//////////////////////////////////////////////////////////////
                              CONSTRUCTOR
   //////////////////////////////////////////////////////////////*/

   constructor() {
       // Owner starts with a little fortune they can distribute.
       _mint(msg.sender, 1_000_000);
   }

   /*//////////////////////////////////////////////////////////////
                             CALLBACK LOGIC
   //////////////////////////////////////////////////////////////*/

  function registerCallback(function (address, address, uint) external callback) external {
      callbacks[msg.sender] = callback;
  }

  function unregisterCallback() external {
      delete callbacks[msg.sender];
  }


   /*//////////////////////////////////////////////////////////////
                              ERC20 LOGIC
   //////////////////////////////////////////////////////////////*/

   function approve(address spender, uint256 amount) public virtual returns (bool) {
       allowance[msg.sender][spender] = amount;

       emit Approval(msg.sender, spender, amount);

       return true;
   }

   function transfer(address to, uint256 amount) public virtual returns (bool) {
       balanceOf[msg.sender] -= amount;

       // Cannot overflow because the sum of all user
       // balances can't exceed the max uint256 value.
       unchecked {
           balanceOf[to] += amount;
       }

       notify(msg.sender, msg.sender, to, amount);
       notify(to, msg.sender, to, amount);

       emit Transfer(msg.sender, to, amount);

       return true;
   }

   function transferFrom(
       address from,
       address to,
       uint256 amount
   ) public virtual returns (bool) {
       uint256 allowed = allowance[from][msg.sender]; // Saves gas for limited approvals.

       if (allowed != type(uint256).max) allowance[from][msg.sender] = allowed - amount;

       balanceOf[from] -= amount;

       // Cannot overflow because the sum of all user
       // balances can't exceed the max uint256 value.
       unchecked {
           balanceOf[to] += amount;
       }

       notify(from, from, to, amount);
       notify(to, from, to, amount);

       emit Transfer(from, to, amount);

       return true;
   }

   function notify(address who, address from, address to, uint amt) internal {
       if (callbacks[who].address != address(0)) {
           callbacks[who](from, to, amt);
       }
   }

   /*//////////////////////////////////////////////////////////////
                       INTERNAL MINT/BURN LOGIC
   //////////////////////////////////////////////////////////////*/

   function _mint(address to, uint256 amount) internal virtual {
       totalSupply += amount;

       // Cannot overflow because the sum of all user
       // balances can't exceed the max uint256 value.
       unchecked {
           balanceOf[to] += amount;
       }

       emit Transfer(address(0), to, amount);
   }

   function _burn(address from, uint256 amount) internal virtual {
       balanceOf[from] -= amount;

       // Cannot underflow because a user's balance
       // will never be larger than the total supply.
       unchecked {
           totalSupply -= amount;
       }

       emit Transfer(from, address(0), amount);
   }
}
```

---

**[Q1] In `transferFrom()`, `unchecked` is __not__ used in the allowance subtraction:**\
(A): To save gas\
(B): To avoid unauthorized transfers\
(C): To avoid reentrancy\
(D): None of the above

 **[Answers]: B**
 
 ---
 
 **[Q2] In `transfer()` and `transferFrom()`, the use of `unchecked` would __not__ be desired:**\
(A): When the token uses large number of decimals\
(B): When the token uses small number of decimals\
(C): At all times\
(D): At no times

**[Answers]: D**

---


**[Q3] In `name()` and `symbol()`, the returned values are incorrect because:**\
(A): The string encoding is too short\
(B): Inline assembly `return` does not leave the function\
(C): `MSTORE` does not fill all bytes until 0x5f and function may return junk at the end\
(D): The code always reverts


**[Answers]: C**

---

**[Q4] To correct `name()`, one could make the following change(s):**\
(A): ```assembly {
          mstore(0x20, 0x20)
          mstore(0x48, 0x0843616c6c6261636b)
          return(0x20, 0x60)
       }```\
(B): ```function name() external pure returns (string memory n) { n = "Callback"; }```\
(C): ```return "Callback";```\
(D): None of the above

**[Answers]: A, B, C**

---


**[Q5] The concern(s) with the check in `notify()` is/are:**\
(A): Selector 0x00000000 is the `fallback` function\
(B): Selector 0x00000000 is the `receive` function\
(C): Selector 0x00000000 is possible in which case a valid callback would not be called\
(D): Selector can never be 0x00000000, so the check is useless

**[Answers]: C**

---

**[Q6] The concern(s) with the call in `notify()` is/are:**\
(A): The call always reverts\
(B): The passed function pointer is internal and therefore not accessible via an external call\
(C): One should always use `try/catch` in external calls\
(D): The called contract may not have the called function selector thus falling through to `fallback` or reverting the transfer

**[Answers]: D**

---

**[Q7] Potential change(s) to `notify()` to mitigate further security concern(s) is/are:**\
(A): Enforce the callback call to use `delegatecall`\
(B): Enforce the callback call to use `staticcall`\
(C): Send Ether to the called contract\
(D): Make the call in inline assembly

**[Answers]: B**

---


**[Q8] How can the contract be exploited for loss-of-funds via notify callback reentrancy?**\
(A): During the callback, while being the sender of a transfer, repeat the transfer\
(B): During the callback, while being the recipient of a transfer, call `transfer` again in the token contract sending the tokens back to the original sender\
(C): During the callback, while being the recipient of a transfer, burn the received tokens\
(D): This cannot happen in this contract

**[Answers]: D**

---
