---
layout: page
title:  "Variables"
permalink: /cairo/examples/variables/
toc: false
---

Variables may be aliased or evaluated:

- Aliased
    - Value **Reference**: `let a = 5`.
    - Expression **Reference**: `let a = x`.
        - An alias may be evaluated with `assert a = b` (checks x equals b).
- Evaluated
    - **Temporary variable**: `tempvar a = 5 * b`.
    - **Local**: `local a = 5 * b`.
    - **Constant**: `const a = 5`.

A variable may be assigned to a function output:

- `let (a) = foo()`.
- `let (local a) = foo()`.

```shell
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

# Constant variables defined here are available to all functions.
# E.g., const my_const_2 = 10

# All persistent state appears in @storage_var
@storage_var
func persistent_state() -> (res : felt):
end

@external
func use_variables{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}():
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

    # Persistent (@storage_var) storage, without a variable.
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

### Monitor

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