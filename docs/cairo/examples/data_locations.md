---
layout: page
title:  "Data locations"
permalink: /cairo/examples/data_locations/
toc: false
---

Data for a variable may exist in one of two locations:

- Storage. Retained in StarkNet state and as CALLDATA on Ethereum.
    - Declared inside a `@storage_var` function.
    - Stored in StarkNet and in the Ethereum blockchain.
- Memory. Transient representation that persists only during some portion
contract execution.
    - Declared within the scope of a contract.
    - Not stored in StarkNet or in the Ethereum blockchain.

```sh
%lang starknet
%builtins pedersen

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.core.storage.storage import Storage
from starkware.cairo.common.alloc import alloc

# Memory
const a_contract_constant = 3

# All persistent state appears in @storage_var
@storage_var
func persistent_state() -> (res : felt):
end

@external
func data_locations{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*}():
    alloc_locals

    # Storage (@storage_var).
    persistent_state.write(80)

    # Memory
    let (an_array : felt*) = alloc()
    assert [an_array] = 9
    assert [an_array + 1] = 8
    local a_tuple : (felt, felt, felt) = (5, 6, 8)
    let a_reference = 15
    tempvar a_temporary = 2 * a_reference
    const a_function_constant = 60
    local a_local = 70
    let (a_state_reference) = persistent_state.read()
    let (local a_local_reference) = persistent_state.read()

    # The function could return any one of the above variables.
    return ()
end

@view
func read_values{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*}(
        ) -> (val_1 : felt, val_2 : felt):
    # Only the Storage is avaliable to someone calling this contract.
    # Of all the variables in this contract, only one is available.
    let (val1) = persistent_state.read()
    let val2 = a_contract_constant
    return (val1, val2)
end
```
Save as `data_locations.cairo`.

### Compile

Then, to compile:
```
starknet-compile data_locations.cairo \
    --output data_locations_compiled.json \
    --abi data_locations_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract data_locations_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x01e805dcdef4a62879af4ffcf5c9efa4087323105b7081f0c97a18213bbe73c3.
Transaction ID: 537434.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=537434

Returns:
{
    "block_id": 25645,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/25645) and the
[contract](https://voyager.online/contract/0x1e805dcdef4a62879af4ffcf5c9efa4087323105b7081f0c97a18213bbe73c3#state)

### Interact

Then, to interact:


```
starknet invoke \
    --network=alpha \
    --address 0x1e805dcdef4a62879af4ffcf5c9efa4087323105b7081f0c97a18213bbe73c3 \
    --abi data_locations_contract_abi.json \
    --function data_locations

Returns:
Invoke transaction was sent.
Contract address: 0x01e805dcdef4a62879af4ffcf5c9efa4087323105b7081f0c97a18213bbe73c3.
Transaction ID: 537469.
```
Call to see what variables are available relative to the invoke function.
```
starknet call \
    --network=alpha \
    --address 0x1e805dcdef4a62879af4ffcf5c9efa4087323105b7081f0c97a18213bbe73c3 \
    --abi data_locations_contract_abi.json \
    --function read_values

Result:
80 3
```
Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.