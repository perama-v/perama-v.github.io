---
layout: page
title:  "Skeleton for SHARP"
toc: false
---

### **Foreword**

Cairo programs directed straight to Ethereum through SHARP
differ from Cairo contracts deployed to StarkNet. This page addresses
SHARP-based programs only. They are the original Cairo program style, and are the most generic,
free-form and flexible.

### **Minimum Verifiable Program**

This is the structure of the most basic Cairo program. Comments are anything with "#" before it,
the rest is code.

```
func main():
    # Code goes here
    return ()
end
```

See [run instructions](../../run_instructions.md) for how to execute this code.

It doesn't do anything, there is no need to prove that.

That could be fun though. If you sent this to a prover, it could generate a proof that this
program (that does nothing) exists and the inputs (none) were used to produce the outputs (none).

The key here is the hash of the program. This is unfakeable. The proof references the hash.

```
# Program hash (of empty skeleton program).
0x46192fc5e7708648336a5e96378d61b176c448c366cba30f9d21b7a35493f60
```

Moreover, the proof references the outputs of the program too. There are none!

### **Program outputs**

Now it's time to add some.

```
# Import output for serialize function.
%builtins output
# Module to send values as an output.
from starkware.cairo.common.serialize import serialize_word

# The output pointer is passed to the function {implicitly}.
# A felt is an integer. A felt* is a pointer to an integer.
func main{output_ptr : felt*}():
    # The number 999 is passed as an output.
    serialize_word(999)
    return ()
end

```

Okay, what does this do? Well, firstly, the program is now different. It has a different hash!

```
# Program hash (of "parrot 999" program).
0x2b34246e5fd6a6340df207ba3ec2bd6ff6b93bd2787bcce496188d27c62a172
```

We have a program, that outputs the number `999`. That is a fact that this program produces,
and a proof can be built to confirm that. The will be saved in the fact registry by the SHARP
proving service, which won't be explained right now.

The important part here is the fact of the matter! As a matter of fact, the program fact starts
with `0x1c7a9`. It is calculated as keccak(program_hash, output_hash). Read more about the hash
calculation [here](https://www.cairo-lang.org/playground-sharp-alpha/). The fact is what
the proof will attest to, it is a unique hash that represents the one-of-a-kind
output/program pair.

```
# python code, join the fun and calculate the fact!
from web3 import Web3
program_hash = 0x2b34246e5fd6a6340df207ba3ec2bd6ff6b93bd2787bcce496188d27c62a172
program_output = [999]
output_hash = Web3.solidityKeccak(['uint256[]'], [program_output])
fact = Web3.solidityKeccak(
    ['uint256', 'bytes32'],
    [program_hash, output_hash])
# fact: 0x1c7a9613bf5780e4b53dd3dee5dd39051dfe6794a24ae567a3541326ef8c5a4b
```

Okay so on Ethereum there may now exist a Solidity contract prepared to confirm that
the program and outputs identifiable by the hash `0x1c7a9...` represent the output of a
real Cairo program. Powerful stuff...

999 confirmed!

### **Fending off Fraud**

But what would that be useful for? Well, not much, but this is progress. Another contract
could have the program hash `0x2b342` saved in storage. A user could come to that contract and make
the claim "the program output is 555, I claim the prize".

That contract, reticent to deliver the prize to anyone who makes a false claim, could do the
following:

```
// Solidity code!
// The contract already knows cairoProgramHash_ = 0x2b342...
// It also knows the verifier contract address (cairoVerifier_)
function claimPrize(uint256[] memory programOutput)
    public
{
    // Get the output hash for the fraudulent 555.
    bytes32 outputHash = keccak256(
        abi.encodePacked(programOutput));
    // Calculate the fact, just like the python version.
    bytes32 fact = keccak256(
        abi.encodePacked(cairoProgramHash_, outputHash));
    // Call the verifier contract (on speed-dial).
    require(
        cairoVerifier_.isValid(fact),
        "MISSING_CAIRO_PROOF");
    // Revert before prize delivery!
    deliverPrize();
}
```

The call to the verifier contract `isValid(fact)` method would return `false`, causing the
Ethereum transaction to revert. There is no fact registered for a hash based on
the fraudulent `555` output, and as such, no prize is delivered.

Well. That DOES seem useful, at least empirically. But anyone looking at the Cairo code
can read the number `999`, and use that to make their claim at the prize. Not so interesting.
It is time to introduce user input.

### **User input**

Users can feed values to a Cairo program. The program can use these inputs for
various tasks, such as executing a transfer or a trade. The program hash does not depend on
the inputs. In the code below, the user creates a `.json` file with their input.

```
{
    "user_id": 5467
}
```

Their ID is already known to the prize contract, so they can put that into the program
to identify themselves.

```
%builtins output
from starkware.cairo.common.serialize import serialize_word

func main{output_ptr : felt*}():
    # Declare that a local variable will be used.
    alloc_locals
    # Make user_id a reference to a local integer.
    local id : felt
    # Run a python hint to access a .json file
    %{
        # python code accesses the user input.
        ids.id = program_input['user_id']
    %}
    # Add the user ID an output.
    serialize_word(id)
    serialize_word(999)
    return ()
end
```

Fantastic. Now anyone who runs this new program will be creating two outputs. For this
user, the output is `[5467, 999]`, which as discussed above, becomes part of the fact
that the proof attests to.

Now the solidity contract can ensure that prize claims are only made by existing users of
the prize contract.

This is how a user can input values into a Cairo program for custom Cairo programs
sent through SHARP. However, user input for programs sent to StarkNet have a different
workflow. This will be discussed elsewhere.
