---
layout: page
title:  "Read and write a tuple"
permalink: /cairo/examples/read_write_tuple/
toc: false
---

Data may be stored as a tuple of values.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

# The tuple to be stored is an output of the storage function.
@storage_var
func stored_felt() -> (res : (felt, felt, felt)):
end

# Function to get the stored tuple.
@view
func get{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}() -> (res_1 : felt, res_2 : felt,
        res_3 : felt):
    let (stored_tuple) = stored_felt.read()
    let res_1 = stored_tuple[0]
    let res_2 = stored_tuple[1]
    let res_3 = stored_tuple[2]
    return (res_1=res_1, res_2=res_2, res_3=res_3)
end

# Function to update the stored tuple of field elements.
@external
func save{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(input_1 : felt, input_2 : felt,
        input_3 : felt):
    # The tuple is declared with round brackets.
    stored_felt.write((input_1, input_2, input_3))
    return ()
end

```
Save as `read_write_tuple.cairo`.

### Compile

Then, to compile:
```
starknet-compile read_write_tuple.cairo \
    --output read_write_tuple_compiled.json \
    --abi read_write_tuple_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract read_write_tuple_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x034695a53e352584f23d5004d7c0f9fffb9147f90cd5bd5348faba5df654e34e
Transaction ID: 28487
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=28487

Returns:
{
    "block_id": 4564,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/4564) and the
[contract](https://voyager.online/contract/0x034695a53e352584f23d5004d7c0f9fffb9147f90cd5bd5348faba5df654e34e#state)

### Interact

Then, to interact:


```
starknet invoke \
    --network=alpha \
    --address 0x34695a53e352584f23d5004d7c0f9fffb9147f90cd5bd5348faba5df654e34e \
    --abi read_write_tuple_contract_abi.json \
    --function save \
    --inputs 50 60 70

Returns:
Invoke transaction was sent.
Contract address: 0x034695a53e352584f23d5004d7c0f9fffb9147f90cd5bd5348faba5df654e34e
Transaction ID: 28917
```


```
starknet call \
    --network=alpha \
    --address 0x34695a53e352584f23d5004d7c0f9fffb9147f90cd5bd5348faba5df654e34e \
    --abi read_write_tuple_contract_abi.json \
    --function get

Returns:
50 60 70
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.