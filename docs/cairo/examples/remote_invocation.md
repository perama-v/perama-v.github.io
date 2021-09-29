---
layout: page
title:  "Remote function invocation"
permalink: /cairo/examples/remote_invocation/
toc: false
---

Draft only.

A function can be called by first passing the function, the later executing it.

This is useful where the function is defined in one location, but arguments or results are
useful in another location. For example, when a contract imports a utility module from
a local file.

The steps are:

- Define the function.
- Get the label of the function.
- Pass the function as an argument.
- Use the `invoke` command.
    - This is the key component.
    - In this example, this will perform storage read and writes
    in the parent function.
- Unpack the values from the `Return` struct.

Folder structure:

    - remote_invocation.cairo
    - remote_invocation_test.py
    - utils
        - numbers.cairo

Main contract: Save as `remote_invocation.cairo`.
```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.registers import get_label_location
from starkware.starknet.common.storage import Storage
from starkware.starknet.common.syscalls import get_caller_address

from utils.numbers import odd_manager

@storage_var
func stored_number() -> (res : felt):
end

@storage_var
func is_odd(user_id : felt) -> (bool : felt):
end

# This function is called by a user.
@external
func is_number_odd{
        storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*,
        syscall_ptr : felt*,
        range_check_ptr
    }():
    # First populate the storage with a test number
    stored_number.write(77)

    # Prepare the two functions.
    let (num_checker) = get_label_location(stored_number.read)
    let (state_writer) = get_label_location(is_odd.write)
    # Pass the functions to the number manager.
    # This might allow the contract to be leaner (if imported).

    # The user will record that they did the number check.
    let (user_id) = get_caller_address()

    # Outsource the work to the util/numbers.cairo contract
    odd_manager(num_checker, state_writer, user_id)

    return ()
end

```


Utility contract: save as `utils/math.cairo`
```sh
%lang starknet

from starkware.cairo.common.alloc import alloc
from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.invoke import invoke
from starkware.cairo.common.math import unsigned_div_rem
from starkware.starknet.common.storage import Storage

# This function will be imported an would not have access to the storage.
# E.g., from util.numbers import odd_manager
func odd_manager{
        storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*,
        syscall_ptr : felt*,
        range_check_ptr
    }(
        num_checker,
        state_writer,
        user_id : felt
    ):
    alloc_locals
    # Note that the two function arguments are not of type 'felt'.

    # The arguments for the invoke function are passed as an array.
    # Create an empty array.
    let (num_checker_args : felt*) = alloc()

    # The manager first retrieves the number-in-question.
    let stored : felt = invoke(
        func_ptr=num_checker,
        n_args=0,
        args=num_checker_args)
    # `Return` is a native struct that contains return values as members
    # whose names are defined in the function (in this case 'value').
    # TODO - use 'stored : Result' - just need the name of the member.

    let (divisor, is_even) = unsigned_div_rem(stored, 2)
    local range_check_ptr = range_check_ptr
    let is_odd = 1 - is_even

    # Now the manager saves the result.
    # Create another argument array
    let (state_writer_args : felt*) = alloc()
    # This time two arguments are passed: the user_id and is_odd.
    assert state_writer_args[0] = user_id
    assert state_writer_args[1] = is_odd

    return ()
end
```

### Compile

Then, to compile:
```
starknet-compile remote_invocation.cairo \
    --output remote_invocation_compiled.json \
    --abi remote_invocation_contract_abi.json
```

### Test

Make a new file called `remote_invocation_test.py` and populate it:

```py
import os
import pytest

from starkware.starknet.compiler.compile import (
    compile_starknet_files)
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

# The path to the contract source code.
CONTRACT_FILE = os.path.join(
    os.path.dirname(__file__), "remote_invocation.cairo")

@pytest.mark.asyncio
async def test_record_items():
    # Compile the contract.
    contract_definition = compile_starknet_files(
        [CONTRACT_FILE], debug_info=True)

    # Create a new Starknet class that simulates the StarkNet
    # system.
    starknet = await Starknet.empty()

    # Deploy the contract.
    contract_address = await starknet.deploy(
        contract_definition=contract_definition)
    contract = StarknetContract(
        starknet=starknet,
        abi=contract_definition.abi,
        contract_address=contract_address,
    )

    # Call the function.
    await contract.is_number_odd().invoke()

```

The packages required for testing are installed by default with cairo-lang.
Run the test:

`pytest remote_invocation_test.py`.

(Work in progress: `AssertionError: Expected memory address to be relocatable value. Found: 0.`)