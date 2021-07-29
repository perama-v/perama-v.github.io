---
layout: page
title:  "Function"
permalink: /cairo/examples/function/
toc: false
---

Functions with a decorator (@view, @external, @storage) only handle arguments of type `felt`.
Other generic helper functions may handle arguments of a type other than `felt`, such as `Struct`,
or pointer.

A contract has two entry points, from which either storage or generic functions may be
accessed.

- @external function for writing.
    - -> @storage to write to state, or generic helper function.
- @view function for reading.
    - -> @storage to read from state, or generic helper function.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage
from starkware.cairo.common.alloc import alloc

struct dataStruct:
    member a : felt
    member b : felt
end

# Only felt arguments
@storage_var
func storage() -> (res : felt):
end

# Only felt arguments
@external
func write{storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(val : felt):
    helper_1(val)
    # Write functions are a transaction and should not return values.
    return ()
end

# Only felt arguments
@view
func read{storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}() -> (val_1 : felt, val_2 : felt,
        val_3 : felt):
    # Brackets around val_1 to receive the value from the helper_2() function.
    let (val_2) = helper_2()
    let (stored_val) = storage.read()
    # Arguments may be passed with their names.
    # If names are not used, they must precede all named arguments (E.g. '9').
    return (9, val_2=val_2, val_3=stored_val)
end

# Other arguments including felt allowed.
func helper_1{storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(val : felt):
    # Implicit arguments in curly brackets when required, such as for
    # storage-related pointers.
    storage.write(val)
    # This function does not return a value.
    return ()
end

# Return values are designated by an arrow '->'
func helper_2() -> (val : felt):
    # This function accepts 0 arguments and returns 1. No implicit arguments needed.
    # 'data' variable is assigned to a struct.
    let data = dataStruct(a=6, b=7)
    # The struct is passed as an argument.
    let (processed_data) = helper_3(data)
    # The use of the name of a return variable E.g., 'val')
    # is optional and here is omitted.
    return (processed_data)
end

func helper_3(a_b_data : dataStruct) -> (processed_data : felt):
    # This function accepts a pointer (to a dataStruct instance),
    # as seen by the '*' asterisk.
    # No implicit arguments needed.
    # A 'tempvar' variable is required for this compound expression.
    tempvar a_b_product = a_b_data.a * a_b_data.b
    return (processed_data=a_b_product)
end
```
Save as `function.cairo`.

### Compile

Then, to compile:
```
starknet-compile function.cairo \
    --output function_compiled.json \
    --abi function_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract function_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x02b6f62e6b42673fb384c2deb77bf9a114a083e99932526537662162eb156497.
Transaction ID: 555287.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=555287

Returns:
{
    "block_id": 26429,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/ID) and the
[contract](https://voyager.online/contract/0x2b6f62e6b42673fb384c2deb77bf9a114a083e99932526537662162eb156497#state)

### Interact

Then, store a value:

```
starknet invoke \
    --network=alpha \
    --address 0x2b6f62e6b42673fb384c2deb77bf9a114a083e99932526537662162eb156497 \
    --abi function_contract_abi.json \
    --function write \
    --inputs 875

Returns:
Invoke transaction was sent.
Contract address: 0x02b6f62e6b42673fb384c2deb77bf9a114a083e99932526537662162eb156497.
Transaction ID: 555317.
```

Then call the read function to access the values with the assistance of the helper functions,
which cannot be called directly.
```
starknet call \
    --network=alpha \
    --address 0x2b6f62e6b42673fb384c2deb77bf9a114a083e99932526537662162eb156497 \
    --abi function_contract_abi.json \
    --function read

Returns:
9 42 875
```
Note that this behaviour matches what is present in the ABI that the compiler produced.
A single write function, and a single read function with three outputs.
```
[
    {
        "inputs": [
            {
                "name": "val",
                "type": "felt"
            }
        ],
        "name": "write",
        "outputs": [],
        "type": "function"
    },
    {
        "inputs": [],
        "name": "read",
        "outputs": [
            {
                "name": "val_1",
                "type": "felt"
            },
            {
                "name": "val_2",
                "type": "felt"
            },
            {
                "name": "val_3",
                "type": "felt"
            }
        ],
        "stateMutability": "view",
        "type": "function"
    }
]

```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.