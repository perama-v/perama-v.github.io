---
layout: page
title:  "Struct"
permalink: /cairo/examples/struct/
toc: false
---

A struct enables arbitrary data types to be defined. The data is not
stored in contract state.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

struct User:
    member id_number : felt
    member is_admin : felt
    member vote_count : felt
end

@storage_var
func admin_votes(user : felt) -> (count : felt):
end

@view
func query_user{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(user_id : felt) -> (value : felt):
    let (votes) = admin_votes.read(user_id)
    return (votes)
end

@external
func register_user{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(id : felt, admin : felt, votes : felt):
    alloc_locals
    # A struct constructor is used to declare member values.
    local new_user : User = User(id_number=id, is_admin=admin,
    vote_count=votes)
    # Struct values can also be changed with:
    # `assert new_user.is_admin = 0`.

    if new_user.is_admin == 1:
        admin_votes.write(new_user.id_number, new_user.vote_count)
        return()
    end
    return ()
end
```
Save as `struct.cairo`.

### Compile

Then, to compile:
```
starknet-compile struct.cairo \
    --output struct_compiled.json \
    --abi struct_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract struct_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x05f6702d9336c4bcddf4f498975ae349753c5f6ecfcd7275c451f19fd17ea429.
Transaction ID: 528495.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=528495

Returns:
{
    "block_id": 25250,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/25250) and the
[contract](https://voyager.online/contract/0x5f6702d9336c4bcddf4f498975ae349753c5f6ecfcd7275c451f19fd17ea429#state)

### Interact

Register a new user with id 8574732 as an admin, with 13 votes:

```
starknet invoke \
    --network=alpha \
    --address 0x5f6702d9336c4bcddf4f498975ae349753c5f6ecfcd7275c451f19fd17ea429 \
    --abi struct_contract_abi.json \
    --function register_user \
    --inputs 8574732 1 13

Returns:
Invoke transaction was sent.
Contract address: 0x05f6702d9336c4bcddf4f498975ae349753c5f6ecfcd7275c451f19fd17ea429.
Transaction ID: 528534.
```
Read the votes:
```
starknet call \
    --network=alpha \
    --address 0x5f6702d9336c4bcddf4f498975ae349753c5f6ecfcd7275c451f19fd17ea429 \
    --abi struct_contract_abi.json \
    --function query_user \
    --inputs 8574732

Returns:
13
```
Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.