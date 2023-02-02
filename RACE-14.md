**Note**: Questions 1 & 2 are based on the below code snippet. You will see the same code snippet for these two questions. The question is below the code snippet.

```solidity
contract USDCCollateral {
  // This is a contract that's part of a larger system implementing a lending protocol
  // This particular contract concerns itself with allowing people to deposit USDC as collateral for their loans

  // A list of all the addresses lending
  address[] lenders;
  // A mapping to allow efficient is lender checks
  mapping(address => bool) isLender;
  
  address immutable lendingPlatform;
  address token = ERC20(USDC_ADDRESS);

  // We use a mapping to store the deposits of all lenders
  mapping (address => bool) balance;

  // USDC is very stable, so for every 1 USDC you can borrow 1 DAI or 1 USD worth of the other currency
  // Similar to other lending platforms, this lending system uses an oracle to determine how much one can borrow. 
  // The following describes how the system determines the max borrow value (ignoring precision for simplicity).
  // maxBorrow = (collateralRatio * underlying * underlyingValueUSD) / otherValueUSD 
  // This encodes the margin requirement of the system.
  uint collateralRatio = 100_000_000; 

  constructor() {
      periodicFee = 1;
      
      // approved collateral contracts are deployed by the lending platform
      // assume it is implemented correctly, and doesn't allow false collateral contracts.
      lendingPlatform = msg.sender;
  }

  function deposit(uint amount) external {
      require(!isLender[msg.sender]);
      isLender[msg.sender] = true;
      lenders.push(msg.sender);

      ...
  }

  function computeFee(uint periodicFee, uint balance) internal returns (uint) {
      // Assume this is a correctly implemented function that computes the share of fees that someone should receive based on their balance.
  }
 
  // this function is called monthly by the lending platform
  // We compute how much fees the lender should receive and send it to them
  function payFees() external onlyLendingPlatform {
      for (uint i=0; i<lenders.length; i++) {
        // Compute fee uses the balance of each lender
         uint fee = computeFee(periodicFee, balance[lenders[i]])
          token.transferFrom(lendingPlatform, lenders[i], fee);
      }
  }
  ...
}
```

**[Q1] Lending platforms have a few options to configure when it comes to adding new tokens as collateral. For example, you’ll have to set up an oracle for the price of the collateral, and you have to configure a margin requirement. The security concern with using the given collateral configuration is:**

(A): The periodic fee parameter is static\
(B): The collateral ratio of loans is too low\
(C): USDC should not be used as collateral for loans\
(D): None of the above

**[Answers]: B**

---

**[Q2] Assuming payFees() is periodically called by a function, which iteratively calls payFees() of all collateral contracts, the security concern(s) is/are:**

(A): Collateral tokens can define their own fee rewards and have the protocol pay too much fees\
(B): There could be many lenders\
(C): payFees() might re-enter the contract, paying all fees again\
(D): You can deposit at any point during the period

**[Answers]: B, D**

---


```solidity
modifier noETH {
  require(balanceOf(this) == 0);
  _;
}
```

**[Q3] The developers want to prevent people from accidentally sending ETH instead of WETH and have implemented a noETH modifier, as defined above, and annotated the deposit function with it. They have also not implemented a receive function. Which of the following statements is true?**

(A): Developers can either use the modifier or achieve the same effect by omitting the payable keyword on deposit function\
(B): Developers should use the modifier because it achieves a different effect from omitting the payable keyword on deposit function\
(C): Developers should remove the modifier but achieve the required effect by omitting the payable keyword on deposit function\
(D): None of the above

**[Answers]: C**

---

```solidity
modifier checkedPool{
  address pool;
  assembly { 
let size := calldatasize()
      pool := calldataload(sub(size, 32))
  }
  require(isValid(pool));
  _;
}

function isValid(pool) internal {...}
function somePoolOperation(...., address pool) public checkedPool { ... }
function anotherPoolOperation(...., address pool) public checkedPool { ... }
```

**[Q4] Developers have used assembly to make their code a bit less repetitive. They use it to annotate a bunch of functions that have as their last argument a pool address. Unfortunately they made a mistake. Which of the following options fixes the bug?**

(A): Replace the modifier with require(isValid(pool)); in every function with the modifier\
(B): Make all functions using checkedPool external\
(C): Everything is fine; this code has no problems\
(D): None of the above

**[Answers]: A**

---

**Note**: Questions 5 & 6 are based on the below code snippet. You will see the same code snippet for these two questions. The question is below the code snippet.

```solidity
interface Lender {
    function onLiquidation() external;
}

interface IERC20 {
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}

contract ERC20Collateral {
    // This contract is different from the one in Questions 1 & 2, please read it carefully!!!
    // This is a contract that's part of a larger system implementing a lending protocol

    // A list of all the addresses lending
    address[] lenders;
    // A mapping to allow efficient is lender checks
    mapping(address => bool) isLender;

    address immutable lendingPlatform = msg.sender;
    IERC20 token;

    // We use a mapping to store the deposits of all lenders
    mapping (address => uint256) balance;

    event Liquidation(address indexed lender, uint256 liquidatedAmount);

    constructor(IERC20 _token) {
        // Consider this contract is now part of a new permissionless factory
    // built by the protocol. The factory behaves very much like a UniswapV2 factory
    // that allows anyone to deploy their own collateral contract
        token = _token;
    }

    function deposit(uint amount) external {
        isLender[msg.sender] = true;
        lenders.push(msg.sender);

        token.transferFrom(msg.sender, lendingPlatform, amount);
        balance[msg.sender] += amount;
    }

    function liquidate(address lender) external {
        // We call the protocol factory to check for under-collateralization conditions
        require(lendingPlatform.undercollateralized(lender, balance[lender]));

        uint256 oldDeposit = balance[lender];
        balance[lender] = 0;
        uint256 fee = oldDeposit / 1000; // fee ratio == 1/1000

        // Give the caller his liquidation execution fee
        token.transferFrom(address(this), msg.sender, fee);
        // Transfer the rest of the collateral to the platform
        token.transferFrom(address(this), lendingPlatform, oldDeposit - fee);
        
        // Now ping the liquidated lender for him to be able to react to the liquidation
        // We need to use a low-level call because the lender might not be a contract
        // and the compiler checks code size on high-level calls, reverting if it's 0
        address(lender).call(abi.encodePacked(Lender.onLiquidation.selector));

        emit Liquidation(lender, oldDeposit - balance[lender]);
    }

    ...
}
```

**[Q5] The lending protocol has also built in a liquidation function to be called in the case of under-collateralization. Anyone can call the function and be rewarded for calling it by taking a small percentage of the liquidation. The liquidation function has a vulnerability  which can be exploited because:**

(A): The lender can open a position with a low amount of collateral and the fee payout reverts\
(B): The lender can make the position “unliquidatable” with reentrancy\
(C): The lender can liquidate other positions with his callback and make more money\
(D): The liquidator can take the full collateral amount with reentrancy

**[Answers]: B**

---

**[Q6] Assume that the vulnerability referenced in the previous question has been fixed by changing the line with the Liquidation event emission to emit Liquidation(lender, oldDeposit). The protocol team has built a bot that monitors and liquidates under-collateralized positions automatically. Even though the bot does not monitor the mempool, it simulates the full transaction and, if successful, sends transactions with the exact amount to be able to execute the function + 100000 gas for minimal execution in the onLiquidation() callback. Which of these attacks can be executed in a harmful way towards the protocol?**

(A): An attacker can liquidate positions, reenter the contract and steal tokens\
(B): The liquidated lender can monitor the mempool and frontrun the protocol bot with a deposit, griefing it\
(C): The lender can make their position “unliquidatable” by consuming all the gas provided in the callback\
(D): The liquidated lender can tokenize the extra gas in the callback and make a profit

**[Answers]: B**

---

**[Q7] In the context of Questions 5 and 6, someone built a MEV frontrunner bot that is exploiting liquidations in different protocols. It monitors the mempool for collateral contracts deployed from the lending factory and simulates transactions in a mainnet fork within Foundry to check whether it should attack them. The logic behind the bot is that it checks only the token’s “Transfer” events for its success conditions. More precisely, it checks if there is liquidity in an AMM to exchange to ETH and make sure it turns a profit at the end. If so, it sends a Flashbot bundle to make a profit by frontrunning the liquidator. Knowing the factory for this new contract is permissionless, how could you extract assets out of this bot?**

(A): Open a position with a low collateral amount to grief the bot\
(B): Build a similar bot that frontruns this one\
(C): Deploy a collateral contract with your own custom token and seed an AMM pool with some ETH and this token, tricking the bot\
(D): There is no way to do it

**[Answers]: C**

---

```solidity
// Assume as adding to the code shown for Questions 5 & 6
contract ERC20Collateral {
  // This is a contract that's part of a larger system implementing a lending protocol
    
    ...

function transferCollateral(
    address signer,
    address receiver,
    uint256 amount,
    bytes32 sigR,
    bytes32 sigS,
    uint8 sigV
  ) external {
    require(signer != address(0), "INVALID_SIGNER");

// Check that the signer is the one who actually signed the message
    require(signer == ecrecover(
    keccak256(abi.encodePacked(receiver, amount)),
      sigV,
      sigR,
      sigS
    ));

    deposits[receiver] += amount;
    deposits[signer] -= amount;
}

    ...
}
```

**[Q8] The protocol implemented a function to transfer collateral from lender A to lender B with a signature from A, as shown above. Is there a way you can break it?**

(A): Lender B can get more than the intended amount from lender A (assuming there is more than double the amount in A’s account)\
(B): Lender A can pretend to transfer to lender B but then steal amount from him\
(C): Lender A can grief lender B by sending a malformed signature (assuming the S parameter is correct)\
(D): Lender B can steal from another lender C, by submitting a malformed signature.

**[Answers]: A**

---
