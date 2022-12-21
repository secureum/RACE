**Note**: All 8 questions in this RACE are based on the below contract. This is the same contract you will see for all the 8 questions in this RACE. The question is below the shown contracts.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";


contract Staking {

    using SafeERC20 for IERC20;

    bool internal _paused;
    address internal _operator;
    address internal _governance;
    address internal _token;
    uint256 internal _minDepositLockTime;

    mapping(address => uint256) _userBalances;
    mapping(address => uint256) _userLastDeposit;

    event Deposit(
        address indexed user,
        uint256 amount
    );

    event Withdraw(
        address indexed user,
        uint256 amount
    );

    constructor(address operator, address governance, address token, uint256 minDepositLockTime) {
        _operator = operator;
        _governance = governance;
        _token = token;
        _minDepositLockTime = minDepositLockTime;
    }

    function depositFor(address user, uint256 amount) external {
        _userBalances[user] += amount;
        _userLastDeposit[user] = block.timestamp;
        
        IERC20(_token).safeTransferFrom(user, address(this), amount);
        
        emit Deposit(msg.sender, amount);
    }

    function withdraw(uint256 amount) external {
        require(!_paused, 'paused');
        require(block.timestamp >= _userLastDeposit[msg.sender] + _minDepositLockTime, 'too early');

        IERC20(_token).safeTransferFrom(address(this), msg.sender, amount);

        if (_userBalances[msg.sender] >= amount) {
            _userBalances[msg.sender] -= amount;
        } else {
            _userBalances[msg.sender] = 0;
        }

        emit Withdraw(msg.sender, amount);
    }

    function pause() external {
        // operator or gov
        require(msg.sender == _operator && msg.sender == _governance, 'unauthorized');
        
        _paused = true;
    }

    function unpause() external {
        // only gov
        require(msg.sender == _governance, 'unauthorized');

        _paused = false;
    }
    
    function changeGovernance(address governance) external {
        _governance = governance;
    }
}
```

---

**[Q1] Which statements are true in `withdraw()`?**

(A) Can be successfully executed when contract is paused       
(B) User can withdraw only after `_minDepositLockTime` elapsed since last withdrawal    
(C) Follows checks-effects-interaction pattern best practice    
(D) User can withdraw more than deposited      
    
**[Answers]: D**    

---

**[Q2] Which mitigations are applicable to `withdraw()`?**

(A): Transferred amount should be minimum of `amount` and `_userBalances[msg.sender]`    
(B): Move if/else block before `safeTransferFrom`    
(C): Require `amount` to be <= user’s balance deposited earlier    
(D): Remove if/else block and add `_userBalances[msg.sender] -= amount` before `safeTransferFrom`    
    
**[Answers]: A, C, D**    

---

**[Q3] The security concern(s) in `pause()` is/are:**    
    
(A): Does not emit an event    
(B): Access control is not strict enough    
(C): Will always revert    
(D): None of the above    
    
**[Answers]: A**    
    
---

**[Q4] Which statement(s) is/are true for `unpause()`?**    
    
(A): Will unpause deposits and withdrawals    
(B): Will unpause withdrawals only    
(C): Anyone can successfully call the function    
(D): None of the above    
    
**[Answers]: B, C**    
    
---

**[Q5] Which statement(s) is/are true in `depositFor()`?**    

(A): Can be executed when contract is paused    
(B): Allows a user to deposit for another user    
(C): Allows a user to fund the deposit for another user    
(D): None of the above    
    
**[Answers]: A, B**    
    
---
    
**[Q6] The issue(s) in `depositFor()` is/are:**    

(A): Cannot be paused for emergency    
(B): Exploitable re-entrancy attack    
(C): User withdrawals can be delayed indefinitely via DoS attack    
(D): None of the above    
    
**[Answers]: A, C**    
    
---
    
**[Q7] Which of the following statement(s) is/are true?**    
    
(A): `Withdraw` event is emitted with incorrect amount    
(B): `Withdraw` event is emitted with correct user    
(C): `Deposit` event is always emitted incorrectly    
(D): `Deposit` event is emitted with incorrect user    
    
**[Answers]: B, D**    
    
---

**[Q8] Potential gas optimization(s) is/are:**    
    
(A): Use `immutable` for all variables assigned in constructor    
(B): Use `immutable` for `_token`, `_operator` and `_minDepositLockTime`    
(C): Use `unchecked`    
(D): None of the above    
    
**[Answers]: B, C**    
    
---
    
