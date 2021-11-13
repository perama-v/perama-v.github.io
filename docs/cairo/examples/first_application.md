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

@storage_var
func balance() -> (res : felt):
end

# Function to get the current balance.
@view
func get{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }() -> (
        value : felt
    ):
    let (value) = balance.read()
    return (value)
end

# Function to increase the balance by 1.
@external
func increase{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }():
    let (res) = balance.read()
    balance.write(res + 1)
    return ()
end

```
Things of note:

- The three implicit arguments in curly brackets are required for
the storage operations.
- When assigning a return value to a reference, capture the
value in brackets: `let (returned_value) = my_func()`
- There is an `@external` function that can modify contract state
and a `@view` function that can only read state. A function
without either of these decorators is only accesible by the contract.

Save as `contracts/first_application.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/first_application.cairo
```

### Test

Make a new file called `test_first_application.py` and populate it:

```py
import pytest
import asyncio
from starkware.starknet.testing.starknet import Starknet

# Enables modules.
@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

# Reusable to save testing time.
@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("contracts/first_application.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Modify contract.
    await contract.increase().invoke()
    await contract.increase().invoke()

    # Read from contract
    response = await contract.get().call()
    assert response.result.value == 2
```
Run the test
```
pytest tests/test_first_application.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy first_application --alias first_application
```

### Interact

Write
```
nile invoke first_application increase
```
Read
```
nile call first_application get
```

### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy first_application --alias first_application --network mainnet
```
Deploymend details
```
🚀 Deploying first_application
🌕 artifacts/first_application.json successfully deployed to 0x02b4cafe5ed8851a5c17ca006810b8848b0306e38f71896c52bdf4e9ef35b16f
📦 Registering deployment as first_application in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online
