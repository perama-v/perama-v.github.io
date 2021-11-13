---
layout: page
title:  "Pytest"
permalink: /cairo/pytest/
toc: false
---

The pytest framework allows for testing of contract deployments
and functions. This is something that comes built into `cairo-lang`
and it is best to see [nile setup]({{ site.baseurl }}{% link cairo/nile.md %})
first to get contract for the framework.

## Minimal Test Template

Make a contract `TEMPLATE.cairo` that has:

- One `@external` function, `EXTERNAL_FUNCTION_NAME()`
    - If should accept two inputs, `INPUT_1` and `INPUT_2`
- One `@view` function, `VIEW_FUNCTION_NAME()`
    - If should return one input, `VAL`.


Make a new file called `TEMPLATE_contract_test.py` and populate it:

```py
import pytest
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

# Make sure that tests start with 'test_'.
@pytest.mark.asyncio
async def test_main_logic():
    # Create the local network
    starknet = await Starknet.empty()
    # Deploy the contract
    contract = await starknet.deploy("contracts/TEMPLATE.cairo")

    # Modify a contract.
    await contract.EXTERNAL_FUNCTION_NAME(INPUT_1, INPUT2).invoke()

    # Read from a contract
    VAL = await contract.VIEW_FUNCTION_NAME().call()
    assert VAL == EXPECTED_RESULT
    print('Value is as expected')
```

The packages required for testing are installed by default with cairo-lang.
Run the test:

```
pytest TEMPLATE_contract_test.py

# Show print statements during testing.
pytest -s TEMPLATE_contract_test.py

# Run a specific test.
pytest TEMPLATE_contract_test.py::test_main_logic
```

## Deployment factory

We move the deployment to a reusable function that two separate
tests can share.

```py
import pytest
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

# Enables modules.
@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

# Reusable to save testing time.
@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("contracts/TEMPLATE.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_main_logic(contract_factory):
    starknet, contract = contract_factory

    # Modify a contract.
    await contract.EXTERNAL_FUNCTION_NAME(INPUT_1, INPUT2).invoke()

    # Read from a contract
    VAL = await contract.VIEW_FUNCTION_NAME().call()
    assert VAL == EXPECTED_RESULT
    print('Value is as expected')

# A second test function that uses the same deployments.
@pytest.mark.asyncio
async def test_two(contract_factory):
    starknet, contract = contract_factory

    VAL = await contract.VIEW_FUNCTION_NAME().call()
    assert VAL == EXPECTED_RESULT
```


## Constructor arguments

If the contract has a `@constructor` function that accepts
arguments on dpeloyment, they are passed as follows:

```
contract = await starknet.deploy(
    "contracts/TEMPLATE.cairo",
    constructor_calldata=[ARG_1, ARG_2]
    )
```

## Account based testing

Accounts in StarkNet differ from Ethereum mainnet in that
StarkNet has Account Abstraction. Every transaction originates from
an account contract

```
# Every user must deploy and initialise an account.
# Initialisation involves saving some key(s), such as a single
# public key.

Account.cairo

# All transactions to application contracts are passed to the
# Account for verification before forwarding on to the application.
```

More about account abstraction in StarkNet can be read
[**here**]({{ site.baseurl }}{% link cairo/account_abstraction.md %})
and
[**here**]({{ site.baseurl }}{% link cairo/account_abstraction_test.md %}).

In the testing framework you can test account based transaction origination,
or you can interact with contracts directly. On Mainnet StarkNet, you
will have to use a account.

The principle for testing with a single-owner account in pytest is
as follows:

- Select an account contract
- Use a helper function to create a `Signer` object that has a
private key
- Deploy the account and save the public key in it
- Determine the payload for your application
- Send a transaction to the account containing a signed payload

One good resource is the OpenZeppelin signer and account

- [signer.py](https://github.com/OpenZeppelin/cairo-contracts/blob/main/tests/utils/Signer.py)
    - Copy this file into a `/tests/utils` directory.
    - Import the signer into pytest file `from test.utils.signer import Signer`
- [Account.cairo](https://github.com/OpenZeppelin/cairo-contracts/blob/main/contracts/Account.cairo)
    - Copy this file into the `/contracts` directory.

How to use an account:
```py
import pytest
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

from utils.Signer import Signer

signer = Signer(123456789987654321)
other = Signer(987654321123456789)

# Enables modules.
@pytest.fixture(scope='module')
def event_loop():
    return asyncio.new_event_loop()

@pytest.fixture(scope='module')
async def contract_factory():
    starknet = await Starknet.empty()
    contract = await starknet.deploy("contracts/TEMPLATE.cairo")
    # Deploy the account
    first_account = await starknet.deploy(
        "contracts/Account.cairo",
        constructor_calldata=[signer.public_key]
    )
    second_account = await starknet.deploy(
        "contracts/Account.cairo",
        constructor_calldata=[other.public_key]
    )
    return starknet, contract, first_account, second_account

@pytest.mark.asyncio
async def test_main_logic(contract_factory):
    starknet, contract, first_account, second_account = contract_factory
    # The signer module will then handle invoking the account contract
    # 'execute()' function and also handle the account nonce.

    await signer.send_transaction(
        account=first_account,
        to=contract,
        selector_name='EXTERNAL_FUNCTION_NAME',
        calldata=[arg_1, arg_2])
    # The application contract can now use the 'get_caller_address()'
    # function from the cairo common library to perform checks on
    # the account making the transaction.
```


