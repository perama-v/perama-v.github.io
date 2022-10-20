---
layout: page
title:  "Solidity skeleton"
permalink: /cairo/examples/solidity_skeleton/
toc: false
---


Here is an example of how a Solidity smart contract can digest and use a proof from a
Cairo program.

This is a short example of a system that involves a smart contract interacting
with a Cairo program. The example highlights how Cairo programs and their proofs
can interact with the Ethereum blockchain.

The user provides a special number to a Cairo program. A proof is generated and
integrated into the Ethereum blockchain. A smart contract checks the proof and
then accepts the special number and performs an action using that number.

## Set up program inputs

Create an ``input.json`` file in the same directory as the cairo code with the following contents.

```
{
    "current_number": 300,
    "special_number": 556
}
```

## Set up Cairo program

Create a file called ``program.cairo`` with the following contents:

```
%builtins output
from starkware.cairo.common.serialize import serialize_word

func main{output_ptr : felt*}():
    alloc_locals
    local original_number : felt  # A local variable
    local a_special_number : felt
    # A hint
    %{
        # Python code that processes the .json file
        num = program_input['current_number']
        ids.original_number = num
        new_num = program_input['special_number']
        ids.a_special_number = new_num
    %}
    serialize_word(original_number)  # Make number an output
    serialize_word(a_special_number)
    return ()
end
```

## Execute program

Now compile program to produce ``program_compiled.json``:

```
cairo-compile program.cairo --output='program_compiled.json'
```

Now run the program, using the compiled ``program_compiled.json`` file:

```
cairo-run --program=program_compiled.json \
    --print_output --print_info --relocate_prints --tracer
```

Confirm that the program output matches the output below:

```
300
556
```

To explore the program structure and debug, visit the tracer at http://localhost:8100/.

## Deploy program

The program can be sent to a public Ethereum testnet (Ropsten) using SHARP. Run the following
command to send the program to SHARP for proof generation and fact registration:

```
cairo-sharp submit --source program.cairo \
    --program_input input.json
```

The above command will produce a ``Fact`` (a hash of the outputs and the program hash). Any Ropsten
application contract or user can now query the ``isValid(Fact)`` read method of the deployed
[Fact Registry](https://ropsten.etherscan.io/address/0xf0EC41069A89595ADf5f27A4a90ff2DF30D83d2E#readContract).

If the result is ``True``, then that application contract or user can be confident that:

-   The Cairo program has computational integrity (**validity**).
-   The inputs used in that program truly produced those outputs (**correctness**).

An application can be built by designing and deploying an Ethereum contract that:

-   Stores the ``program_hash`` permanently to be able to recognise this unique Cairo program.
-   Accepts the outputs that come from that Cairo program.
-   Uses those values in some way.

## Application Design

That application contract needs to have a method that performs particular steps.
The steps and some corresponding Solidity examples are outlined below for a function
called ``updateState()``, which:

Accepts, as an argument, the Cairo program outputs `programOutput`.

``function updateState(uint256[] memory programOutput) public {}``.

Computes the output hash, `outputHash`.

``bytes32 outputHash = keccak256(abi.encodePacked(programOutput));``.

Computes the `fact`, which is a keccak hash of the Pedersen hash of the program
and the program outputs. The program hash is permanent and can be retrieved from
contract storage.

``bytes32 fact = keccak256(abi.encodePacked(cairoProgramHash_, outputHash));``.

Calls the `Fact Registry` read method ``isValid(fact)`` to determine if the proof
should be accepted. The address of the verifier is permanent and can be
retrieved from contract storage.

``require(cairoVerifier_.isValid(fact), "MISSING_CAIRO_PROOF");``.

Updates the application state `applicationState_`, accessing the program outputs by index,
according to the specific application.

``applicationState_ = programOutput[1]``.

In this way, a user may interact with a Cairo program to ultimately execute a change on Ethereum
without using large amounts of expensive storage or computation.

## Application Deployment

A solidity contract ``CairoApplication.sol`` can be deployed to use the fact in the
``FactRegistry`` contract for its logic. That contract may have a function ``updateState()``
which can be called by passing outputs from ``program.cairo`` as arguments. The function
call would be a transaction that updates the state of the application to reflect the
state changes that the Cairo program created.

## Application Use

A user, or agent on behalf of a user, can interact with the application by following the steps:

1. Create an ``inputs.json`` file with custom values.
2. Obtain a copy of ``program.cairo``.
3. Install Cairo and compile the program.
4. Submit the program to SHARP for proving.
5. Call ``updateState()`` function in the ``CairoApplication`` contract.

These steps can be abstracted away from the user experience with the use of an interface and
server backend.

## Solidity code

Below is the Solidity code referenced in the above descriptions.

When the contract is deployed, the `constructor` will need to be passed values that will
permanently live in the contract:
- `cairoProgramHash`, the enshrined Cairo program hash that the application will recognise.
- `cairoVerifier`, the address of the Ropsten verifier, deployed by Starkware
(`0xf0EC41069A89595ADf5f27A4a90ff2DF30D83d2E`).
- `initialNumber`, the "first number" the application will store.

This can be organised through smart contract deployment software such as
[hardhat](https://hardhat.org/getting-started/#quick-start). These variables might be stored
in the `deploy.js` that hardhat will use to send the contract to Ropsten.

```
pragma solidity ^0.5.2;

contract IFactRegistry {
    /*
    Returns true if the given fact was previously registered.
    */
    function isValid(bytes32 fact)
        external view
        returns(bool);
}

contract SpecialNumber {

    // The current special number
    uint256 currentNumber_;

    // The Cairo program hash.
    uint256 cairoProgramHash_;

    // The Cairo verifier.
    IFactRegistry cairoVerifier_;

    constructor(
        uint256 cairoProgramHash,
        address cairoVerifier,
        uint256 initialNumber)
        public
    {
        currentNumber_ = initialNumber;
        cairoProgramHash_ = cairoProgramHash;
        cairoVerifier_ = IFactRegistry(cairoVerifier);
    }

    function updateNumber(uint256[] memory programOutput)
        public
    {
        // Ensure that a corresponding proof was verified.
        bytes32 outputHash = keccak256(
            abi.encodePacked(programOutput));
        bytes32 fact = keccak256(
            abi.encodePacked(cairoProgramHash_, outputHash));
        require(
            cairoVerifier_.isValid(fact),
            "MISSING_CAIRO_PROOF");

        // Ensure the output consistency with currentstate.
        require(
            programOutput.length == 2,
            "INVALID_PROGRAM_OUTPUT");
        require(
            currentNumber_ == programOutput[0],
            "MISSING_ORIGINAL_NUMBER");
        require(
            currentNumber_ != programOutput[1],
            "NUMBER_MUST_BE_DIFFERENT");

        // Update stored number with new number.
        currentNumber_ = programOutput[1];
    }
}
```

## Caveats

This application is meant to illustrate the mechanics of an application,
rather than to show the true power of using Cairo to increase throughput
on Ethereum.

A more sophisticated application might involve the Cairo program accepting Ethereum
addresses so that the proof could contain a record of who submitted a number. To
to explore how scaling might be achieved, the app might instead accept multiple numbers. The
Cairo program could then accept a collection of users and their numbers, thereby increasing the
density of a proof from one number to many numbers. The proof cost would practically be the
same cost and the cost per user would be a lot less.
