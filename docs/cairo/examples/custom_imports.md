---
layout: page
title:  "Custom Imports"
permalink: /cairo/examples/custom_imports/
toc: false
---

If you have a function that is reusable, you can store it in a
separate `.cairo` file and then import it. This can be helpful to
make contracts more readable and modular.


Consider an application that has an addition function.
```
- contracts/
    - custom_imports.cairo
    - utils/
        - math.cairo
```

```sh
%lang starknet
%builtins range_check
# contracts/custom_imports.cairo

# Import from teh cairo-lang package
from starkware.cairo.common.cairo_builtins import HashBuiltin
# Import a function from a custom local file.
from contracts.utils.math import add_two, get_modulo

# This is the main contract file that will be deployed.
@view
func get_calculations{
        range_check_ptr
    }(
        first : felt,
        second : felt
    ) -> (
        sum : felt,
        modulo : felt
    ):
    # Two custom operations.
    let (sum) = add_two(first, second)
    let (modulo) = get_modulo(first, second)
    return (sum, modulo)
end

```
Save the contract above as `contracts/custom_imports.cairo`

Now save the module to be imported (below) as `contracts/utils/math.cairo`.
Nile will deploy the main contract and bring the required functions
in during compilation. Only one contract is deployed.

```sh
%lang starknet


# This function is from the common library.
# It can live here, reducing the clutter of an application.
from starkware.cairo.common.math import unsigned_div_rem

# This function is imported by 'custom_import.cairo'
# Equivalent to placing this function in that file.
func add_two(
        a : felt,
        b : felt
    ) -> (
        sum : felt
    ):
    let sum = a + b
    return (sum)
end

# This function performs the modulo operation.
func get_modulo{
        range_check_ptr
    }(
        a : felt,
        b : felt
    ) -> (
        result : felt
    ):
    let (dividend, remainder) = unsigned_div_rem(a, b)
    # The dividend is not used, and the following is equivalent:
    # let (_, remainder) = unsigned_div_rem(a, b)
    return (remainder)
end
```

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/custom_imports.cairo
```

### Test

Make a new file called `test_custom_imports.py` and populate it:

```py
import pytest
import asyncio
from starkware.starknet.testing.starknet import Starknet

@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    # Note how the contracts/utils/math.cairo file is not needed here.
    contract = await starknet.deploy("contracts/custom_imports.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    num_1 = 10
    num_2 = 3
    # Read from contract
    response = await contract.get_calculations(10, 3).call()
    # Check the results, addressing each returned value
    # by its name defined in the contract return statement.
    assert response.result.sum == num_1 + num_2
    assert response.result.modulo == num_1 % num_2

```
Run the test
```
pytest tests/test_custom_imports.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy custom_imports --alias custom_imports
```

### Interact

Read
```
nile call custom_imports get_calculations 10 3
```
Returns: `13 1`


### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy custom_imports --alias custom_imports --network mainnet
```
Returns:

```
🚀 Deploying custom_imports
🌕 artifacts/custom_imports.json successfully deployed to 0x0030cfe7438b17ccf64584e4f76f038c1f7f647dc85eedf4be1b175c8d7a7d2b
📦 Registering deployment as custom_imports in mainnet.deployments.txt
```
Deployments can be viewed in the voyager explorer
https://voyager.online
