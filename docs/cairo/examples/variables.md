---
layout: page
title:  "Variables"
permalink: /cairo/examples/variables/
toc: false
---

There are two types of variables in Cairo contracts:

- **Transient (Non-State)**
    - Declared inside a `@view` or `@external` function.
    - Not stored in StarkNet or in the Ethereum blockchain.
    - Further divided into four:
        - Revocable
            - **Reference**:`let a = 5` (felt only).
            - **Temporary variable**: `tempvar a = 5 * b` (expressions).
        - Non-revocable
            - **Constant**: `const a = 5` (felt only, known at compile time)
            - **Local**: `local a = 5 * b` (expressions, defined during contract interaction).
- **Persistent (State)**
    - Declared inside a `@storage_var` function.
    - Stored in StarkNet and in the Ethereum blockchain.

Global variables that access blockchain data are not available as of Cairo v0.2.0.

```shell
%lang starknet
%builtins pedersen

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.core.storage.storage import Storage

# Constant variables defined here are available to all functions.
# E.g., const my_const_2 = 10

# All persistent state appears in @storage_var
@storage_var
func persistent_state() -> (res : felt):
end

@external
func use_variables{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*}():
    alloc_locals  # needed for "local" variables.

    # Transient, revocable felt (reference).
    let my_reference = 50
    # Redefine reference.
    let my_reference = 51

    # Transient, revocable expression (temporary variable).
    tempvar my_tempvar = 2 * my_reference
    # Redefine tempvar.
    tempvar my_tempvar = 3 * my_reference

    # Transient, non-revocable felt (constant).
    const my_const = 60
    # Cannot redefine const (const my_const = 61)

    # Transient, non-revoacable expression (local). Requires alloc_locals.
    local my_local = 70
    # Cannot redefine local (local my_local = 71).

    # Persistent (@storage_var).
    persistent_state.write(80)
    # Redefine state.
    persistent_state.write(81)

    # A variable can be assigned to a function output:
    # let (my_var) = func().
    let (my_reference_2) = persistent_state.read()
    let (local my_local_2) = persistent_state.read()

    return ()
end

```
Save as `variables.cairo`.

### Compile

Then, to compile:
```
starknet-compile variables.cairo \
    --output variables_compiled.json \
    --abi variables_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract variables_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x06fac555b6910a5a778a0913ddc8234c1cd855bcd8fd2e57c728234a45bb171f.
Transaction ID: 497984.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=497984

Returns:
{
    "block_id": 23906,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/23906) and the
[contract](https://voyager.online/contract/0x6fac555b6910a5a778a0913ddc8234c1cd855bcd8fd2e57c728234a45bb171f)

### Interact

Then, to interact:

```
starknet invoke \
    --network=alpha \
    --address 0x6fac555b6910a5a778a0913ddc8234c1cd855bcd8fd2e57c728234a45bb171f \
    --abi variables_contract_abi.json \
    --function use_variables

Returns:
Invoke transaction was sent.
Contract address: 0x6fac555b6910a5a778a0913ddc8234c1cd855bcd8fd2e57c728234a45bb171f.
Transaction ID: 497985.
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.