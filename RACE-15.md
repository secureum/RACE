**Note:** All 8 questions in this RACE are based on the below contract. This is the same contract you will see for all the 8 questions in this RACE. The question is below the shown contract.

```solidity
// SPDX-License-Identifier: agpl-3.0
pragma solidity ^0.8.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.0.0/contracts/token/ERC20/IERC20.sol";

contract SimpleDEX {
   uint64 public token_balance;
   uint256 public current_eth;
   IERC20 public token;
   uint8 public reentrancy_lock;
   address owner;
   uint256 public fees;
   uint256 public immutable fees_percentage = 10;

   modifier nonReentrant(){
     // Logic needs to be implemented
       _;
   }

   modifier onlyOwner(){
       require(tx.origin == owner, "Only owner permitted");
       _;
   }

    constructor(uint first_balance, address _token, address _owner) payable {
        require(_owner != address(0) , "No zero Address allowed");
        require(msg.value >= 100);
        token = IERC20(_token);
        bool token_success = token.transferFrom(msg.sender, address(this), first_balance);
        require (token_success, "couldn't transfer tokens");
        owner = _owner;
        token_balance = uint64(first_balance);
        current_eth = msg.value;
    }

   function setOwner(address _owner) public onlyOwner {
       owner = _owner;
   }

   function getTokenBalance() public view returns(uint64 _token_balance) {
       _token_balance = token_balance;
   }

   function getCurrentEth() public view returns(uint256 _current_eth) {
       _current_eth = current_eth;
   }

   function getEthPrice() public view returns(uint) {
       return uint256(token_balance) / current_eth;
   }

   function claimFees() public onlyOwner {
       bool token_success =  token.transfer(msg.sender, fees);
       require(token_success, "couldn't transfer tokens");
       token_balance -= uint64(fees);
       fees = 0;
   }

   function buyEth(uint amount) external nonReentrant {
       require(amount >= 10);
       uint ratio = getEthPrice();
       uint fee = (amount / 100) * fees_percentage;
       uint token_amount = ratio * (amount + fee);
       bool token_success = token.transferFrom(msg.sender, address(this), token_amount);
       current_eth -= amount;
       require(token_success, "couldn't transfer tokens");
       (bool success, ) = msg.sender.call{value: amount}("");
       require(success, "Failed to transfer Eth");
       token_balance += uint64(token_amount);
       fees += ratio * fee;
   }

   fallback() payable external {
       revert();
   }
}


contract SimpleDexProxy {
   function buyEth(address simpleDexAddr, uint amount) external {
       require(amount > 0, "Zero amount not allowed");
       (bool success, ) = (simpleDexAddr).call(abi.encodeWithSignature("buyEth(uint)", amount));
       require (success, "Failed");
   }
}


contract Seller {
        // Sells tokens to the msg.sender in exchange for eth, according to SimpleDex's getEthPrice()
        function buyToken(SimpleDEX simpleDexAddr) external  payable {
            uint ratio = simpleDexAddr.getEthPrice();
            IERC20 token = simpleDexAddr.token();
            uint256 token_amount = msg.value * ratio;
            token.transfer(msg.sender, token_amount);
        }
}
```

---

**[Q1] What is/are the correct implementation(s) of the `nonReentrant()` modifier?** \
(A):

```solidity
       require (reentrancy_lock == 1);
       reentrancy_lock = 0;
        _;
       reentrancy_lock = 1;
```

(B):

```solidity
       require (reentrancy_lock == 0);
       reentrancy_lock = 1;
        _;
       reentrancy_lock = 0;
```

(C):

```solidity
       require (reentrancy_lock == 1);
       reentrancy_lock = 1;
        _;
       reentrancy_lock = 0;
```

(D):

```solidity
       require (reentrancy_lock == 0);
       reentrancy_lock = 2;
       _;
       reentrancy_lock = 0;
```

<details><summary><b>[Answers]</b></summary><b>
B, D
</b></details>

---

**[Q2] Who can claim fees using `claimFees()`?** \
(A): Only the owner, due to `onlyOwner` modifier \
(B): The owner \
(C): Anyone who can trick owner into signing an arbitrary transaction \
(D): No one

<details><summary><b>[Answers]</b></summary><b>
B, C
</b></details>

---

**[Q3] In `buyEth()`, we put an `unchecked` block on “`current_eth -= amount`”:** \
(A): Because `current_eth` is `uint` \
(B): Because the compiler is protecting us from overflows \
(C): Only if we add a prior check: \
 `require(current_eth > amount);` \
(D): Only if we add a prior check: \
 `require(current_eth >= amount);`

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q4] In `buyEth()`, are there any reentrancy concerns assuming the `nonReentrant` modifier is implemented correctly?** \
(A): No, because it has the `nonReentrant` modifier \
(B): No, and even without the modifier you can't exploit any issue \
(C): Yes, there is a cross-contract reentrancy concern via `Seller` \
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
C
</b></details>

---

**[Q5] What will happen when calling `buyEth()` via `SimpleDexProxy`?** \
(A): `buyEth()` will be called and successfully executed \
(B): You can’t call a function that way; it must be called directly \
(C): `buyEth()` will be called but ETH won't be transferred \
(D): Transaction will be reverted

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q6] In `buyEth()`:** \
(A): If `amount` is less than 100, it will lead to an incorrect calculation of `fee` \
(B): If `token_balance` is already at its `MAX_UINT256`, it will result in overflow and won't revert \
(C): If `token_amount` is > `MAX_UINT64`, it will result in a casting issue \
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A, C
</b></details>

---

**[Q7] Can `getEthPrice()` return zero?** \
(A): Yes, if the owner initializes the contract with more ETH than `token_balance` \
(B) Yes, a carefully crafted `buyEth()` transaction can result in `getEthPrice()` returning zero \
(C): Yes, once all the ETH are sold \
(D): No, there is no issue

<details><summary><b>[Answers]</b></summary><b>
A
</b></details>

---

**[Q8] Which of the following invariants (written in propositional logic) hold on a correct implementation of the code?** \
(A): `this.balance == current_eth` <=> `token.balanceOf(this) == token_balance` \
(B): `this.balance >= current_eth` && `token.balanceOf(this) >= token_balance` \
(C): `this.balance <= token.balanceOf(this)` && `token.balanceOf(this) <= token_balance` \
(D): `this.balance >= current_eth` || `token.balanceOf(this)  >= token_balance`

<details><summary><b>[Answers]</b></summary><b>
B, D
</b></details>

---
