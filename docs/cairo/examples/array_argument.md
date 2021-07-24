---
layout: page
title:  "Array as transaction argument"
permalink: /cairo/examples/array_argument/
toc: false
---

A call to an `@external` function may accept an array as an argument.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

@storage_var
func stored_felt() -> (res : felt):
end

# Function to get the stored value.
@view
func get{storage_ptr : Storage*, pedersen_ptr : HashBuiltin*, range_check_ptr}(
        ) -> (res : felt):
    let (stored_solution) = stored_felt.read()
    return (stored_solution)
end

# Function to accept an array.
@external
func save{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*, range_check_ptr}(
        input_array_len : felt, input_array : felt*):
    let first = input_array[0]
    let last = input_array[input_array_len - 1]
    let solution = first * 2 + last * 3
    stored_felt.write(solution)
    return ()
end

```
Save as `array_argument.cairo`.

### Compile

Then, to compile:
```
starknet-compile array_argument.cairo \
    --output array_argument_compiled.json \
    --abi array_argument_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract array_argument_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x029caa6660d24ec2a2676c49b942d06f4fabf3f6525dc530ccaef5814afd96fb
Transaction ID: 30455
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=30455

Returns:
{
    "block_id": 4916,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/4916) and the
[contract](https://voyager.online/contract/0x29caa6660d24ec2a2676c49b942d06f4fabf3f6525dc530ccaef5814afd96fb#state)

### Interact

Then, to interact:

```
starknet invoke \
    --network=alpha \
    --address 0x29caa6660d24ec2a2676c49b942d06f4fabf3f6525dc530ccaef5814afd96fb \
    --abi array_argument_contract_abi.json \
    --function save \
    --inputs 5 1 2 3 4 5

Returns:
Invoke transaction was sent.
Contract address: 0x029caa6660d24ec2a2676c49b942d06f4fabf3f6525dc530ccaef5814afd96fb
Transaction ID: 47065
```

Test that the contract has stored the sum using the array correctly.

```
starknet call \
    --network=alpha \
    --address 0x29caa6660d24ec2a2676c49b942d06f4fabf3f6525dc530ccaef5814afd96fb \
    --abi array_argument_contract_abi.json \
    --function get

Returns:
17
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.