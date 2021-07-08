---
layout: page
title:  "Hello, World!"
permalink: /cairo/examples/hello_world/
toc: false
---


`%lang starknet` tells the compiler that this is a contract,
rather than a Cairo program.
```sh
%lang starknet

@view
func echo_world(number : felt) -> (res : felt):
    return (res=number)
end
```
Save as `hello_world.cairo`.
### Compile

Then, to compile:
```
starknet-compile hello_world.cairo \
    --output hello_world_compiled.json \
    --abi hello_world_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract hello_world_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address:  0x06df56691fd57868034e2d48489e35859888d86659f748fec8c26db19b21d6dd.
Transaction ID: 497953.
```
### Interact

Invoke the echo_world function and send the number
`884422` as an input:

```
starknet invoke \
    --network=alpha \
    --address \
    0x06df56691fd57868034e2d48489e35859888d86659f748fec8c26db19b21d6dd \
    --abi hello_world_contract_abi.json \
    --function echo_world \
    --inputs 884422

Returns:
Invoke transaction was sent.
Contract address: 0x06df56691fd57868034e2d48489e35859888d86659f748fec8c26db19b21d6dd.
Transaction ID: 497954.
```

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=497954

Returns:
{
    "block_id": 23889,
    "tx_status": "ACCEPTED_ONCHAIN"
}
```
The [block](https://voyager.online/block/23889) and the
[contract](https://voyager.online/contract/0x6df56691fd57868034e2d48489e35859888d86659f748fec8c26db19b21d6dd#state)

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.




Visit the [voyager explorer](https://voyager.online/) to see the transactions.