---
layout: page
title:  "Skeleton for StarkNet"
toc: false
---

### **Foreword**

The Cairo language may be used to write custom programs and custom contracts.

Most developers will use StarkNet contracts to write applications because
they offer composability with other L2 applications.

- Cairo program (proof-based applications)
    - Programs that may be compiled and sent to SHARP
    - Generate 'facts' on mainnet
    - Used for highly customizable applications
    - Are the backend of StarkNet OS, which powers StarkNet
- Cairo contract (L2 blockchain applications)
    - Cairo program that is passed to the StarkNet OS
    - Has a subset Cairo language available
    - Used for programs that interact with each other on L2 StarkNet

StarkNet contracts are just like Solidity contracts.
They are stateful, can be deployed and interacted with, and
and exist inside blocks chained together. Cairo contracts are stapled
back to Ethereum as aggregate STARK proofs that summarise key state updates.
Proofs make state data available on Ethereum, and a Solidity
contract can verify proofs to police StarkNet state. StarkNet is a
an L2 in the sense that it inherets the security of Ethereum.

Main diferences between a Cairo program and a StarkNet contract.

- Storage means that Cairo contracts can read and write values for access
in between different Cairo contract invocations.
- Decorators prepend functions and change their scope/behaviour.
- ABI (application binary interface). A convention that a user will
user to specify inputs.
- "No hints" means that Cairo contracts don't allow StarkNet operators
the priveilege of running python code.
- No `main()`. Well, users are calling functions through the ABI now,
so no need for a single entry point!

### **Minimum Verfifiable Contract**

So what do Cairo contracts look like? Solidity contracts! Sort of.

This is a template for a `demo_contract.cairo` file.

```
# Declare that this is a StarkNet contract, not a Cairo program.
%lang starknet

# Think Solidity: constructor
@storage_var
func value() -> (res : felt):
    # Creates a variable that will persist in contract storage.
end

# Think Solidity: public
@external
func update_val():
    # can call value.read() and value.write().
    return ()
end

# Think Solidity: public view
@view
func read_val():
    # can call value.read().
    return ()
end
```

For comparison, here is a template for a `DemoContract.sol` solidity
file. The `value` variable is initialised, and after contract
deployment, its value can be read by `readVal()`, or modified by
`writeVal()`.


```
// Solidity analogue to the above Cairo contract.
contract DemoContract {
    bytes32 value;

    // Think Cairo: @storage_var
    constructor(bytes32 _value) {
        value = _value;
    }

    // Think Cairo: @external
    function writeVal() public {
    }

    // Think Cairo: @view
    function readVal() public view {
    }
}

```

When `starknet-compile` is run for the skeleton Cairo contract above, `contract_abi.json`
is created. This contains the interface that should be used when interacting with the contract
after it is deployed to StarkNet.

It can be seen that the two functions are indeed part of the interface, and that `read_val`
has `stateMutability` set to `view`, reflecting that this function cannot modify contract
storage.

```
[
    {
        "inputs": [],
        "name": "update_val",
        "outputs": [],
        "type": "function"
    },
    {
        "inputs": [],
        "name": "read_val",
        "outputs": [],
        "stateMutability": "view",
        "type": "function"
    }
]
```
### **Contract Deployment**

When `starknet-deploy` is run for the compiled contract, the address that the contract is
deployed to is returned, along with a transaction ID:

```
Contract address: 0x00c43c9ec02c60b5bbdf7cb67004ca1ecc4a61c5862a67473019b6a7d455b315.
Transaction ID: 358737.
```

Fantastic! Now the network is holding onto the less-than-useful demonstration contract.
The non-functional `read_val()` method can be called.

```
starknet invoke \
    --address 0x00c43c9ec02c60b5bbdf7cb67004ca1ecc4a61c5862a67473019b6a7d455b315 \
    --abi contract_abi.json \
    --function read_val \
    --inputs
```
Success!
```
Invoke transaction was sent.
Contract address: 0x00c43c9ec02c60b5bbdf7cb67004ca1ecc4a61c5862a67473019b6a7d455b315.
Transaction ID: 358747.
```
Calling with `--inputs 1` raises an error because the function was defined to accept
0 arguments. This is also true of the demo `update_val()` function.
```
Error: AssertionError: Wrong number of arguments. Expected 0, got 1.
```

```
starknet get_code --contract_address \
0x00c43c9ec02c60b5bbdf7cb67004ca1ecc4a61c5862a67473019b6a7d455b315

```

A more expansive contract example can be found in the
[StarkNet docs](https://www.cairo-lang.org/docs/hello_starknet/intro.html).