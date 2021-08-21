---
layout: page
title:  "Pedersen hash"
permalink: /cairo/examples/pedersen_hash/
toc: false
---

The Pedersen hash of two elements `x` and `y` can be found using the `hash2(x, y)` module from the
Cairo common library.


```sh
%lang starknet
%builtins pedersen

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.hash import hash2

@view
func get_hash{pedersen_ptr: HashBuiltin*}(x, y) -> (
        hash: felt , hash_with_zero: felt):
    # Pedersen hash of x and y.
    # The hash function used will be the pointer that is passed.
    let (hash) = hash2{hash_ptr=pedersen_ptr}(x, y)

    # Hash of x and 0. Equivalent to the hash of x alone.
    let (hash_with_zero) = hash2{hash_ptr=pedersen_ptr}(x, 0)

    return (hash, hash_with_zero)
end

```
Save as `pedersen_hash.cairo`.

### Compile

Then, to compile:
```
starknet-compile pedersen_hash.cairo \
    --output pedersen_hash_compiled.json \
    --abi pedersen_hash_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract pedersen_hash_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x02fe395f2c3467b6458017e5ff69fd992f46b75b8efb919f97b6724906bfa235
Transaction ID: 182130
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=182130

Returns:
{
    "block_id": 41371,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/ID) and the
[contract](https://voyager.online/contract/0x2fe395f2c3467b6458017e5ff69fd992f46b75b8efb919f97b6724906bfa235#state)

### Interact

Then, to interact:


```
starknet call \
    --network=alpha \
    --address 0x2fe395f2c3467b6458017e5ff69fd992f46b75b8efb919f97b6724906bfa235 \
    --abi pedersen_hash_contract_abi.json \
    --function get_hash \
    --inputs 13579 123

Returns:
1991472738936771294314932046141177383402302165974022666190392465136523753950

2255487090060981340412778270523682108366421523848318719210969003889439916982
```

This is the hash of (13579, 123) and (13579, 0) respectively.

### Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.