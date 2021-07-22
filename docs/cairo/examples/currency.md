---
layout: page
title:  "Currency"
permalink: /cairo/examples/currency/
toc: false
---

There is no sense of a native currency in Cairo. Any variable can be used
to create a system of ownership of layer-1 ether or tokens.

The contract would need to check that the sender is the owner. Below is a
wallet system that does not check if the sender of a transaction has the
authority to control a given wallet.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

@storage_var
func wallet_balance(user : felt) -> (res : felt):
end

@external
func register_currency{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(
        user : felt, register_amount : felt):
    alloc_locals
    let (local balance) = wallet_balance.read(user)
    wallet_balance.write(user, balance + register_amount)
    return ()
end

@external
func move_currency{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(
        from_user : felt, to_user : felt, move_amount : felt):
    alloc_locals
    let (local sender_balance) = wallet_balance.read(from_user)
    let (local receiver_balance) = wallet_balance.read(to_user)
    wallet_balance.write(to_user, receiver_balance + move_amount)
    wallet_balance.write(from_user, sender_balance - move_amount)
    return ()
end

@view
func check_wallet{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(
        user : felt) -> (balance : felt):
    alloc_locals
    let (local balance) = wallet_balance.read(user)
    return (balance)
end
```
Save as `currency.cairo`.

### Compile

Then, to compile:
```
starknet-compile currency.cairo \
    --output currency_compiled.json \
    --abi currency_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract currency_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x05d4d79c959ded1b278139f6c9c79a828296c92626af93634ed8359f944cbad7.
Transaction ID: 502536.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=502536

Returns:
{
    "block_id": 24116,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/24116) and the
[contract](https://voyager.online/contract/0x5d4d79c959ded1b278139f6c9c79a828296c92626af93634ed8359f944cbad7#state)

### Interact

Register that user 7 owns 100 units currency, which might represent mainnet ether (ETH).

```
starknet invoke \
    --network=alpha \
    --address 0x5d4d79c959ded1b278139f6c9c79a828296c92626af93634ed8359f944cbad7 \
    --abi currency_contract_abi.json \
    --function register_currency \
    --inputs 7 100

Returns:
Invoke transaction was sent.
Contract address: 0x05d4d79c959ded1b278139f6c9c79a828296c92626af93634ed8359f944cbad7.
Transaction ID: 502591.
```

Move 2 ether from user 7 to user 789:
```
starknet invoke \
    --network=alpha \
    --address 0x5d4d79c959ded1b278139f6c9c79a828296c92626af93634ed8359f944cbad7 \
    --abi currency_contract_abi.json \
    --function move_currency \
    --inputs 7 789 2

Returns:
Invoke transaction was sent.
Contract address: 0x05d4d79c959ded1b278139f6c9c79a828296c92626af93634ed8359f944cbad7.
Transaction ID: 502617.
```
Check that the currency was received:
```
starknet call \
    --network=alpha \
    --address 0x5d4d79c959ded1b278139f6c9c79a828296c92626af93634ed8359f944cbad7 \
    --abi currency_contract_abi.json \
    --function check_wallet \
    --inputs 789

Returns:
2
```
Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.