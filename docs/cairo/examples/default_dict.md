---
layout: page
title:  "Dictionary"
permalink: /cairo/examples/default_dict/
toc: false
---

A dictionary is created using the `default_dict_new()` module from
the Common Library. Every key will return the default value unless
it is explicitly set.

Steps:

1. Get a pointer to a new dictionary with `default_dict_new()`.
2. Assert its integrity with `default_dict_finalize()`.
3. Assign a value to a key `dict_write()`.
4. Read the value of a key `dict_read()`.


```sh
%lang starknet
%builtins range_check

from starkware.cairo.common.default_dict import (
    default_dict_new, default_dict_finalize)
from starkware.cairo.common.dict import (DictAccess,
    dict_write, dict_read, dict_update)

# Returns the value for the specified key in a dictionary.
@view
func get_value_of_keys{range_check_ptr}(
        key_1 : felt, key_2 : felt, key_3 : felt) -> (
        val_1 : felt, val_2 : felt, val_3 : felt):
    alloc_locals

    # First create an empty dictionary and finalize it.
    # All keys will initially have value 13. {key: 13}.
    let initial_value = 13
    let dict_ptr : DictAccess* = default_dict_new(
        default_value=initial_value)
    # At first, the dict has one pointer (start==end).
    # Save the start_pointer of the dictionary for later.
    local dict_start : DictAccess* = dict_ptr

    # Finalize the dictionary. This ensures default value is correct
    # in the event you want to read the val of a non-existent key.
    let (_, dict_ptr) = default_dict_finalize(
        dict_accesses_start=dict_start,
        dict_accesses_end=dict_ptr,
        default_value=initial_value)

    # Check that an unused key returns the default value.
    # We have called finalize() at least once, so the default val
    # cannot be modified by a malicious prover.
    # Note that the unused key cannot be called as an input to this
    # contract.
    let (local unused_key_999_val) = dict_read{dict_ptr=dict_ptr}(
        key=999)
    assert unused_key_999_val = 13

    # Then add {key: val} pairs.
    # Passing dict as an implicit argument allows the function
    # to automatically move the pointer to the end of the dict.
    dict_write{dict_ptr=dict_ptr}(key=4, new_value=17)  # {4: 17}.
    # dict_ptr is rebound to the end of the dict every write.

    dict_write{dict_ptr=dict_ptr}(key=10, new_value=6)  # {10: 6}.
    # ^^ dict_ptr is again redefined. Using the specific term
    # 'dict_ptr' allows this. Otherwise you would write:
    # 'let my_dict_end = dict_ptr' after every write.

    # Finalize the dictionary. This ensures default value is correct
    # in the event you want to read the val of a non-existent key.

    let (_, dict_ptr) = default_dict_finalize(
        dict_accesses_start=dict_start,
        dict_accesses_end=dict_ptr,
        default_value=initial_value)


    # Check {key: value} pair is correct.
    let (key_4_val) = dict_read{dict_ptr=dict_ptr}(key=4)
    assert key_4_val = 17

    # Update a key. {4: 17} becomes {4: 18}
    dict_update{dict_ptr=dict_ptr}(key=4,
        prev_value=17, new_value=18)

    # Call finalize() if modifications were made since the last read.
    let (_, dict_ptr) = default_dict_finalize(
        dict_accesses_start=dict_start,
        dict_accesses_end=dict_ptr,
        default_value=initial_value)

    let (local val_1) = dict_read{dict_ptr=dict_ptr}(key_1)
    let (local val_2) = dict_read{dict_ptr=dict_ptr}(key_2)
    let (local val_3) = dict_read{dict_ptr=dict_ptr}(key_3)
    return (val_1, val_2, val_3)
end
```
Save as `default_dict.cairo`.

### Compile

Then, to compile:
```
starknet-compile default_dict.cairo \
    --output default_dict_compiled.json \
    --abi default_dict_contract_abi.json
```
### Test

Make a new file called `default_dict_contract_test.py` and populate it:

```py
import pytest
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

@pytest.mark.asyncio
async def test_database():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("default_dict.cairo")

    # Call a function.
    res = await contract.get_value_of_keys(
        2, 4, 10).call()

    # Check the result of a function.
    assert res == (13, 18, 6)
```

The packages required for testing are installed by default with cairo-lang.
Run the test:

`pytest default_dict_contract_test.py`.

### Deploy

Then, to deploy:
```
starknet deploy --contract default_dict_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x048f4f8dcc102f7efdba57c9aadc0cd95b5531844e8a0152568f492e7746aa3f
Transaction ID: 269359
```

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=269359

Returns:
{
    "block_id": 41489,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/41489) and the
[contract](https://voyager.online/contract/0x048f4f8dcc102f7efdba57c9aadc0cd95b5531844e8a0152568f492e7746aa3f#state)

### Interact

Then, to interact:

```
starknet call \
    --network=alpha \
    --address 0x048f4f8dcc102f7efdba57c9aadc0cd95b5531844e8a0152568f492e7746aa3f \
    --abi default_dict_contract_abi.json \
    --function get_value_of_keys \
    --inputs 2 4 10

Returns:
13 18 6
```

Key-value pairs returned are:

- `{2: 13}` A default value.
- `{4: 18}` the key that
was assigned and then updated.
- `{10: 6}` the key that was assigned.

### Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.