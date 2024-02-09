**Note:** All 8 questions in this RACE are based on the below contract. This is the same contract you will see for all the 8 questions in this RACE. The question is below the shown contract.
```solidity
// SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/interfaces/IERC3156FlashLender.sol";
import "@openzeppelin/contracts/interfaces/IERC20.sol";

contract FlashLoan is IERC3156FlashLender {
   bytes32 public constant CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");
   uint256 public fee;

   /**
    * @param fee_ the fee that should be paid on a flashloan
    */
   constructor (
       uint256 fee_
   ) {
       fee = fee_;
   }

   /**
    * @dev The amount of currency available to be lent.
    * @param token The loan currency.
    * @return The amount of `token` that can be borrowed.
    */
   function maxFlashLoan(
       address token
   ) public view override returns (uint256) {
       return IERC20(token).balanceOf(address(this));
   }

   /**
    * @dev The fee to be charged for a given loan.
    * @param token The loan currency. Must match the address of this contract.
    * @param amount The amount of tokens lent.
    * @return The amount of `token` to be charged for the loan, on top of the returned principal.
    */
   function flashFee(
       address token,
       uint256 amount
   ) external view override returns (uint256) {
       return fee;
   }

   /**
    * @dev Loan `amount` tokens to `receiver`, and takes it back plus a `flashFee` after the ERC3156 callback.
    * @param receiver The contract receiving the tokens, needs to implement the `onFlashLoan(address user, uint256 amount, uint256 fee, bytes calldata)` interface.
    * @param token The loan currency. Must match the address of this contract.
    * @param amount The amount of tokens lent.
    * @param data A data parameter to be passed on to the `receiver` for any custom use.
    */
   function flashLoan(
       IERC3156FlashBorrower receiver,
       address token,
       uint256 amount,
       bytes calldata data
   ) external override returns (bool){
       uint256 oldAllowance = IERC20(token).allowance(address(this), address(receiver));
       uint256 oldBal = IERC20(token).balanceOf(address(this));
       require(amount <= oldBal, "Too many funds requested");
       IERC20(token).approve(address(receiver), oldAllowance + amount);

       require(
           receiver.onFlashLoan(msg.sender, token, amount, fee, data) == CALLBACK_SUCCESS,
           "Callback failed"
       );

       uint256 newBal = IERC20(token).balanceOf(address(this));
       if(newBal < oldBal + fee) {
           uint retAmt = oldBal + fee - newBal;
           require(IERC20(token).transferFrom(msg.sender, address(this), retAmt), "All funds not returned");
       }

       if (IERC20(token).allowance(address(this), address(receiver)) > oldAllowance) {
           IERC20(token).approve(address(receiver), oldAllowance);
       }
       return true;
   }
}
```
---
**[Q1] Which of the following is an explanation of why `flashLoan()` could revert?** \
(A): The transaction reverts because a user requested to borrow more than `maxFlashLoan()` \
(B): The transaction reverts because the receiver’s `onFlashLoan()` did not return `CALLBACK_SUCCESS` \
(C): The transaction reverts because the user returned more than `retAmt` funds \
(D): The transaction reverts because a user tried to spend more funds than their allowance in `onFlashLoan()`

**[Answers]: A, B, D**

---

**[Q2] If the `FlashLoan` contract were safe, which of the following invariants should hold at the end of any given transaction for some ERC20 token t? Note: old(expr) evaluates expr at the beginning of the transaction.** \
(A): t.balanceOf(address(this)) >= old(t.balanceOf(address(this))) \
(B): t.balanceOf(address(this)) == old(t.balanceOf(address(this))) \
(C): t.balanceOf(address(this)) > old(t.balanceOf(address(this))) \
(D): t.balanceOf(address(this)) == old(t.balanceOf(address(this))) + fee

**[Answers]: A**

---

**[Q3] Which of the following tokens would be unsafe for the above contract to loan as doing so could result in theft?** \
(A): ERC223 \
(b): ERC677 \
(B): ERC777 \
(C): ERC1155 
    
**[Answers]:  C**

---

**[Q4] Which external call made by `flashLoan()` could result in theft if the token(s) identified in the previous question  were to be used?** \
(A): `onFlashLoan()` \
(B): `balanceOf()` \
(C): `transferFrom()` \
(D): `approve()` 

**[Answers]:  C**

---

**[Q5] What is the purpose of the fee in the `FlashLoan` contract as is?** \
(A): To increase the size of available flashloans over time \
(B): To pay the owner of the flashloan contract \
(C): To pay those who staked their funds to be flashloaned \
(D): It has no purpose

**[Answers]:  A** 

---

**[Q6] [Q6] Which of the following describes the behavior of `maxFlashLoan` for a standard ERC20 token over time?** \
(A): Strictly-increasing \
(B): Non-decreasing \
(C): Constant \
(D): None of the above

**[Answers]:  B**

---

**[Q7] For some arbitrary ERC20 token t, which of the following accurately describes the `FlashLoan` contract's balance of t after a successful (i.e. non-reverting) call to `flashLoan()` (where t is the token requested for the flashloan)?** \
(A): The `FlashLoan` contract's balance of token t will INCREASE OR STAY THE SAME \
(B): The `FlashLoan` contract's balance of token t will DECREASE OR STAY THE SAME \
(C): The `FlashLoan` contract's balance of token t will STAY THE SAME \
(D): None of the above

**[Answers]:  D**

---

**[Q8] Which of the following are guaranteed to hold after a successful (i.e., non-reverting) execution of `flashLoan()`, assuming the token for which the flashloan is requested uses OpenZeppelin’s Standard ERC20 implementation?** \
(A): The receiver’s balance of “token” increases \
(B): The funds that the `FlashLoan` contract has approved the `receiver` to spend has either stayed the same or decreased \
(C): The sum of all flashloans granted by the `FlashLoan` contract is less than the `maxFlashLoan` amount \
(D): The token balance of any contract/user other than the `FlashLoan` contract, the caller of the `flashLoan()`, and the “receiver” contract will remain the same as before the call to `flashLoan()`

**[Answers]:  B**  

---
