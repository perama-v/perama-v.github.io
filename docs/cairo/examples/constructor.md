---
layout: page
title:  "Constructor"
permalink: /cairo/examples/constructor/
toc: false
---

When a contract is deployed to StarkNet, if it has a special
`@constructor` function then this function will be run once. It
can not be run after deployment.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func stored_parameter() -> (res : felt):
end

@storage_var
func important_contract_address() -> (res : felt):
end

# Run on deployment only. Must have constructor in name and decorator.
@constructor
func constructor{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
      special_number : felt,
      important_address : felt
    ):
    stored_parameter.write(special_number)
    important_contract_address.write(important_address)
    return ()
end

# Function to get the numbers stored at deployment.
@view
func read_special_values{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }() -> (
        number : felt,
        address : felt
    ):
    let (number) = stored_parameter.read()
    let (address) = important_contract_address.read()
    return (number, address)
end


```
Save as `contracts/constructor.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/constructor.cairo
```

### Test

Make a new file called `test_constructor.py` and populate it:

In pytest, the deployment arguments are passed as follows:

```py
contract = await starknet.deploy("contracts/constructor.cairo",
    constructor_calldata=[ARG_1, ARG_2])
```
Full test code:
```py
import pytest
import asyncio
from starkware.starknet.testing.starknet import Starknet

PARAMETER = 23456
IMPORTANT_ADDRESS = 987623451345

# Enables modules.
@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

# Reusable to save testing time.
@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("contracts/constructor.cairo",
        constructor_calldata=[PARAMETER, IMPORTANT_ADDRESS])
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Read from contract
    response = await contract.read_special_values().call()
    assert response.result == (PARAMETER, IMPORTANT_ADDRESS)
```
Run the test
```
pytest tests/test_constructor.py
```

### Local Deployment

Deploy to the local devnet.
```
# Not currently enabled in nile.

# Option 1
nile deploy constructor 345 876 --alias constructor

# Option 2 Separate inputs flag (conflicts with the invoke/call pattern)
nile deploy constructor --inputs 345 876 --alias constructor

# Option 3 (option 1 with automatic aliasing of contract name)
nile deploy constructor 345 876
```

### Interact

Read
```
nile call constructor read_special_values
```


### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy constructor --alias constructor --network mainnet
```
Deployments can be viewed in the voyager explorer
https://voyager.online
