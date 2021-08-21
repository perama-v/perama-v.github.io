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
from starkware.cairo.common.dict import (
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
    let (local dict) = default_dict_new(default_value=initial_value)
    # Finalize the dictionary. This ensures default value is correct.
    default_dict_finalize(
        dict_accesses_start=dict,
        dict_accesses_end=dict,
        default_value=initial_value)

    # Then add {key: val} pairs.
    dict_write{dict_ptr=dict}(key=4, new_value=17)  # {4: 17}.
    dict_write{dict_ptr=dict}(key=10, new_value=6)  # {10: 6}.

    # Check {key: value} pair is correct.
    let (key_4_val) = dict_read{dict_ptr=dict}(key=4)
    assert key_4_val = 17

    # Update a key.
    dict_update{dict_ptr=dict}(key=4,
        prev_value=17, new_value=18)  # {4: 17} -> {4: 18}

    # Check that an unused key returns the default value.
    let (unused_key_999_val) = dict_read{dict_ptr=dict}(
        key=999)
    assert unused_key_999_val = 13

    # Get value of the requested keys.
    let (val_1) = dict_read{dict_ptr=dict}(key_1)
    let (val_2) = dict_read{dict_ptr=dict}(key_2)
    let (val_3) = dict_read{dict_ptr=dict}(key_3)
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
### Deploy

Then, to deploy:
```
starknet deploy --contract default_dict_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x0239d703aabc8f9202a3c9c7b2f6969bdb0af23981fcb21624135c3c32d006d4
Transaction ID: 198291
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=198291

Returns:
{
    "block_id": 44334,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/44334) and the
[contract](https://voyager.online/contract/0x239d703aabc8f9202a3c9c7b2f6969bdb0af23981fcb21624135c3c32d006d4#state)

### Interact

Then, to interact:

```
starknet call \
    --network=alpha \
    --address 0x239d703aabc8f9202a3c9c7b2f6969bdb0af23981fcb21624135c3c32d006d4 \
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