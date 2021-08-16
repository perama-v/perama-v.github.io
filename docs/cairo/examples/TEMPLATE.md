---
layout: page
title:  "TEMPLATE"
permalink: /cairo/examples/TEMPLATE/
toc: false
---


DESCRIPTION

```sh
CONTRACT

```
Save as `TEMPLATE.cairo`.

### Compile

Then, to compile:
```
starknet-compile TEMPLATE.cairo \
    --output TEMPLATE_compiled.json \
    --abi TEMPLATE_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract TEMPLATE_compiled.json \
    --network=alpha

Returns:
RESULT
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=ID

Returns:
RESULT
```
The [block](https://voyager.online/block/ID) and the
[contract](https://voyager.online/contract/ADDRESS#state)

### Interact

Then, to interact:


```
starknet invoke \
    --network=alpha \
    --address ADDRESS \
    --abi TEMPLATE_contract_abi.json \
    --function FUNCTION_NAME \
    --inputs INPUT1 INPUT2

Returns:
RESULT
```

### Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.