---
layout: page
title:  "First Application"
permalink: /cairo/examples/first_application/
toc: false
---


In this contract a balance is stored. A call may increase
the balance.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

@storage_var
func balance() -> (res : felt):
end

# Function to get the current balance.
@view
func get{storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}() -> (res : felt):
    let (res) = balance.read()
    return (res)
end

# Function to increase the balance by 1.
@external
func increase{storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}():
    let (res) = balance.read()
    balance.write(res + 1)
    return ()
end

```
Save as `first_application.cairo`.

### Compile

Then, to compile:
```
starknet-compile first_application.cairo \
    --output first_application_compiled.json \
    --abi first_application_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract first_application_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x02f445f5e40064ee3df87767add1c202ead56e3f3040b7a3ec27f814616ac402.
Transaction ID: 497971.
```
*Remove the leading zero* 0x02f becomes 0x2f

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=497971

Returns:
{
    "block_id": 23893,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/23893) and the
[contract](https://voyager.online/contract/0x2f445f5e40064ee3df87767add1c202ead56e3f3040b7a3ec27f814616ac402#state)

### Interact

Then, to interact with the external function:

```
starknet invoke \
    --network=alpha \
    --address 0x2f445f5e40064ee3df87767add1c202ead56e3f3040b7a3ec27f814616ac402 \
    --abi first_application_contract_abi.json \
    --function increase

Returns:
Invoke transaction was sent.
Contract address: 0x02f445f5e40064ee3df87767add1c202ead56e3f3040b7a3ec27f814616ac402.
Transaction ID: 497972.
```

Then, interact with the view function:

```
starknet call \
    --network=alpha \
    --address 0x2f445f5e40064ee3df87767add1c202ead56e3f3040b7a3ec27f814616ac402 \
    --abi first_application_contract_abi.json \
    --function get

Returns:
4
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.




Visit the [voyager explorer](https://voyager.online/) to see the transactions.