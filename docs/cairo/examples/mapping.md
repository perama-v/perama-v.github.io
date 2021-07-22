---
layout: page
title:  "Mapping"
permalink: /cairo/examples/mapping/
toc: false
---

Maps are created with storage functions with one or more parameter fields.
Nested maps are created by adding more parameters:

- `func map_a(key : felt) -> (value : felt)`.
    - Equivalent to  `map_a[key]`.
- `func map_b(key1 : felt, key2) -> (value : felt)`.
    - Equivalent to `map_b[key1][key2]`.


```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

# A single value.
@storage_var
func store_a() -> (count : felt):
end

# A single mapping of key -> value.
# Specific id_number -> count.
@storage_var
func store_b(id_number : felt) -> (count : felt):
end

# Nested mapping of key1 and key2 -> value
# Specific item size and item price combination -> count.
# Within a storage type (large_cheap, small_expensive).
@storage_var
func store_c(size : felt, price : felt) -> (count : felt):
end

@external
func record_inventory{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(a_val : felt, b_id : felt, b_val : felt,
        c_size : felt, c_price : felt, c_val : felt):

    store_a.write(a_val)
    store_b.write(b_id, b_val)
    store_c.write(c_size, c_price, c_val)
    return ()
end

@view
func read_inventory{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(b_id : felt, c_size : felt,
        c_price : felt) -> (storage_a : felt, storage_b : felt,
        storage_c : felt):
    alloc_locals
    # Read the requested parts of each inventory.
    # "let (local xyz)" = func() pattern is used for function return values.

    let (local count_a) = store_a.read()
    let (local count_b) = store_b.read(b_id)
    let (local count_c) = store_c.read(c_size, c_price)
    return (count_a, count_b, count_c)
end

```
Save as `mapping.cairo`.

### Compile

Then, to compile:
```
starknet-compile mapping.cairo \
    --output mapping_compiled.json \
    --abi mapping_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract mapping_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x03408bba79d86ab45ee5ee2b93f36c7fac8786baf6f05d6a666d1d9aa239c86d.
Transaction ID: 497986.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=497986

Returns:
{
    "block_id": 23910,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/23910) and the
[contract](https://voyager.online/contract/0x3408bba79d86ab45ee5ee2b93f36c7fac8786baf6f05d6a666d1d9aa239c86d#state)

### Interact

Then, to interact, set the inputs to match the order they appear. In this case,
recording the inventory of

- 100 type A goods
- 3030 type B goods (with ID 37)
- 44 type C goods that are size 9889, and price $12.

```
a_val b_id b_val c_size c_price c_val
100   37   3030  9889   12      44
```

```
starknet invoke \
    --network=alpha \
    --address 0x3408bba79d86ab45ee5ee2b93f36c7fac8786baf6f05d6a666d1d9aa239c86d \
    --abi mapping_contract_abi.json \
    --function record_inventory \
    --inputs 100 37 3030 9889 12 44

Returns:
Invoke transaction was sent.
Contract address: 0x03408bba79d86ab45ee5ee2b93f36c7fac8786baf6f05d6a666d1d9aa239c86d.
Transaction ID: 497987.
```

```
starknet tx_status --network=alpha --id=497987

Returns:
{
    "block_id": 23911,
    "tx_status": "PENDING"
}
```
Read the inventory, specifying the nature of the items from b and c that
were stored in the last transaction:
```
b_id c_size c_price
37   9889   12
```
Where invoke requires a transaction, call does not.
```
starknet call \
    --network=alpha \
    --address 0x3408bba79d86ab45ee5ee2b93f36c7fac8786baf6f05d6a666d1d9aa239c86d \
    --abi mapping_contract_abi.json \
    --function read_inventory \
    --inputs 37 9889 12

Returns:
100 3030 44
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.
