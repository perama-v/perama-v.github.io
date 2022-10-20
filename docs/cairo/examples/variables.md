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

# Constant variables defined here are available to all functions.
# E.g., const my_const_2 = 10

# All persistent state appears in @storage_var
@storage_var
func persistent_state() -> (res : felt):
end

@external
func use_variables{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }():
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

    # Transient, non-revocable expression (local). Requires alloc_locals.
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

    assert my_local_2 = 81

    return ()
end

```

Things to note:

- Locals are good when you want to assert that a variable has
a certain value. If you use a reference with an assert statement,
the reference is redefined (it assigns rather than checks). By using
a local, the contract would perform the check and halt with an error.

Save as `contracts/variables.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/variables.cairo
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy variables --alias variables
```

### Interact

Read
```
nile call variables use_variables
```


### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy variables --alias variables --network mainnet
```
```
🚀 Deploying variables
🌕 artifacts/variables.json successfully deployed to 0x025dd2b60d1439b9e65a972930b8c5a35b3fdb0f50d523abdf3faa87dd42c79d
📦 Registering deployment as variables in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online
