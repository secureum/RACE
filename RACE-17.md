**Note**: All 8 questions in this RACE are based on the below contract. This is the same contract you will see for all the 8 questions in this RACE. The question is below the shown contract.
```
pragma solidity 0.8.19;


import {Pausable} from "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/Pausable.sol";
import {IERC20} from "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol";
import {SafeERC20} from "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol";


interface IWETH is IERC20 {
  function deposit() external payable;
  function withdraw(uint256) external;
}


contract Vault is Pausable {
  using SafeERC20 for IWETH;


  address public immutable controller;
  IWETH public immutable WETH;


  // about 4700 ETH
  uint72 public constant TOTAL_CONTRIBUTION_CAP = type(uint72).max;
  // ALLOWANCE_CAP = 40% of TOTAL_CONTRIBUTION_CAP
  uint256 public immutable ALLOWANCE_CAP = 40 * uint256(TOTAL_CONTRIBUTION_CAP) / 100;
  uint72 public totalContributions;
  mapping (address => uint72) individualContributions;


  uint256 numContributors;
  event ContributorsUpdated(address newContributor, uint256 indexed oldNumContributors, uint256 indexed newNumContributors);


  constructor(address _controller, IWETH _weth) {
      controller = _controller;
      WETH = _weth;
  }


  function deposit(uint72 amount) external payable whenNotPaused {
      if (msg.value > 0) {
          WETH.deposit{value: amount}();
      } else {
          WETH.transferFrom(msg.sender, address(this), amount);
      }
      require((totalContributions += amount) <= TOTAL_CONTRIBUTION_CAP, "cap exceeded");
      if (individualContributions[msg.sender] == 0) emit ContributorsUpdated(msg.sender, numContributors, numContributors++);
      individualContributions[msg.sender] += amount;
  }


  function withdraw(uint72 amount) external whenNotPaused {
      individualContributions[msg.sender] -= amount;
      totalContributions -= amount;
      // unwrap and call
      WETH.withdraw(amount);
      (bool success, ) = payable(address(msg.sender)).call{value: amount}("");
      require(success, "failed to transfer ETH");
  }


  function requestAllowance(uint256 amount) external {
      // ALLOWANCE_CAP is 40% of TOTAL_CAP
      uint256 allowanceCap = ALLOWANCE_CAP;
      uint256 allowance = amount > totalContributions ? allowanceCap : amount;
      WETH.safeApprove(controller, allowance);
  }


  // for unwrapping WETH -> ETH
  receive() external payable {
      require(msg.sender == address(WETH), "only WETH contract");
  }
}
```
---
**[Q1] deposit() can revert:** \
(A): If insufficient ETH was sent with the call \
(B): If the caller has insufficient WETH balance \
(C): With the "cap exceeded" error \
(D): If called internally 

**[Answers]: A, B**

---
**[Q2] What issues pertain to the `deposit()` function?** \
(A): Funds can be drained through re-entrancy \
(B): Accounting mismatch if user specifies `amount` > `msg.value` \
(C): Accounting mismatch if user specifies `amount` < `msg.value` \
(D): `totalContributionCap` isn't enforced on an individual level 

**[Answers]: C**

---
**[Q3] Which of the following is/are true about `withdraw()`?** \
(A): Funds can be drained through re-entrancy \
(B): Funds can be drained due to improper amount accounting in `deposit()` \
(C): Assuming a sufficiently high gas limit, the function reverts from the recipient (caller) consuming all forwarded gas \
(D): May revert with "failed to transfer ETH"

**[Answers]: D**

---
**[Q4] Which of the following parameters are correctly emitted in the `ContributorsUpdated()` event?** \
(A): `newContributor` \
(B): `oldNumContributors` \
(C): `newNumContributors` \
(D): None of the above

**[Answers]: A**

---
**[Q5] The vault deployer can pause the following functions:** \
(A): `deposit()` \
(B): `withdraw()` \
(C): `requestAllowance()` \
(D): None of the above

**[Answers]: D**

---
**[Q6] What is the largest possible allowance given to the controller?** \
(A): 40% of `totalContributionCap` \
(B): 60% of `totalContributionCap` \
(C): 100% of `totalContributionCap` \
(D): Unbounded

**[Answers]: C**

---
**[Q7] The `requestAllowance()` implementation would have failed after the 1st call for tokens that only allow zero to non-zero allowances. Which of the following mitigations do NOT work?** \
(A): `safeApprove(0)` followed by `safeApprove(type(uint256).max)` \
(B): `safeIncreaseAllowance(type(uint256).max)` \
(C): `safeIncreaseAllowance(0)` followed by `safeIncreaseAllowance(type(uint256).max)` \
(D): `safeDecreaseAllowance(0)` followed by `safeApprove(type(uint256).max)` 

**[Answers]: B, C, D**

---
**[Q8] Which of the following gas optimizations are relevant in reducing runtime gas costs for the vault contract?** \
(A): Changing `ALLOWANCE_CAP` type from immutable to constant, ie. `uint256 public constant ALLOWANCE_CAP = 40 * uint256(TOTAL_CONTRIBUTION_CAP) / 100;` \
(B): Increase number of solc runs (assuming default was 200 runs) \
(C): Renaming functions so that the most used functions have smaller method IDs \
(D): Use `unchecked` math in `withdraw()`

**[Answers]: B, C**

---
