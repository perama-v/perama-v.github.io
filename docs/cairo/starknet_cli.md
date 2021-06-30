---
layout: page
title:  "StarkNet CLI"
permalink: /cairo/starknet-cli/
toc: false
---

The StarkNet command line interface (CLI) is used to interact with a `contract.cairo`
file locally, or a contract already deployed to StarkNet. The full CLI implementation
is
[here](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/starknet/cli/starknet_cli.py).

Options:

- Compile a contract with [starknet-compile](#starknet-compile).
- Deploy a contract with [starknet deploy](#starknet-deploy).
- Write to a function with [starknet invoke](#starknet-invoke).
- Read the value of a function with [starknet call](#starknet-call).
- Monitor a submitted transaction with [starknet tx_status](#starknet-tx_status).
- Get the contents of a block with [starknet get_block](#starknet-get_block).
- Get the bytecode of a block with [starknet get_code](#starknet-get_code).
- Get the value of a storage variable with [starknet get_storage_at](#starknet-get_storage_at).

## starknet-compile

```
starknet-compile contract.cairo \
    --output contract_compiled.json \
    --abi contract_abi.json
```
This is special in that the command is hyphenated.
The contract to be compiled is specified, as is the output
file names for both the compiled contract and the ABI.


## starknet deploy

`starknet deploy --contract compiled_contract.json`

The compiled contract is sent to StarkNet for deployment.

*Optional*

`starknet deploy --contract compiled_contract.json --address SELECTED_ADDRESS`

An optional address `SELECTED_ADDRESS` specifying where the contract will be deployed.
If the address is not specified, the contract will be deployed in a random address.

## starknet invoke

```
starknet invoke \
    --address CONTRACT_ADDRESS \
    --abi abi.json \
    --function FUNCTION \
    --inputs INPUT_1 INPUT_2 INPUT_3 \
```

A transaction is sent to StarkNet calling the `FUNCTION` of the contract specified
using the inputs as arguments. This command can update a contract state.

## starknet call

```
starknet call \
    --address CONTRACT_ADDRESS \
    --abi abi.json \
    --function FUNCTION \
    --inputs INPUT_1 INPUT_2 INPUT_3
```

A transaction is sent to StarkNet calling the `FUNCTION` of the contract specified
using the inputs as arguments. This command is read-only.

*Optional*

```
starknet call \
    --address CONTRACT_ADDRESS \
    --abi abi.json \
    --function FUNCTION \
    --inputs INPUT_1 INPUT_2 INPUT_3 \
    --block_id BLOCK_ID
```

Passing the `BLOCK_ID` will execute the call in the context of that block. If not specified,
the context used is the latest block.

## starknet tx_status
```
starknet tx_status \
    --id TX_ID
```

Queries the status of a transaction given its ID.

Possible return values:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
- PENDING: The transaction passed the validation and is waiting to be sent on-chain.
- REJECTED: The transaction failed validation and thus was skipped.
- ACCEPTED_ONCHAIN: The transaction was accepted on-chain.

## starknet get_block

`starknet get_block`

Outputs the latest block.

*Optional*

```
starknet get_block \
    --id BLOCK_ID
```

Outputs the specified block.

## starknet get_code

```
starknet get_code \
    --contract_address CONTRACT_ADDRESS
```

Gets the bytecode of the specified contract at the latest block.

*Optional*

```
starknet get_code \
    --contract_address CONTRACT_ADDRESS \
    --block_id BLOCK_ID
```
Gets the bytecode of the specified contract at the specified block.

## starknet get_storage_at

```
starknet get_storage_at \
    --contract_address CONTRACT_ADDRESS \
    --key KEY
```

Gets the storage variable of the specified contract at the latest block.

The key is calculated using python and the variable name (E.g., 'balance'):
```
from starkware.starknet.public.abi import get_storage_var_address

balance_key = get_storage_var_address('balance')
```

*Optional*

```
starknet get_storage_at \
    --contract_address CONTRACT_ADDRESS \
    --key KEY \
    --block_id BLOCK_ID
```

Gets the storage variable of the specified contract at the specified block.
