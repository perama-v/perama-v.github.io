---
layout: page
title:  "Currency"
permalink: /cairo/examples/currency/
toc: false
---

There is no sense of a native currency in Cairo. Any variable can be used
to create a system of ownership of layer-1 ether or tokens.

The contract would need to check that the sender is the owner. Below is a
wallet system that does not check if the sender of a transaction has the
authority to control a given wallet.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

@storage_var
func wallet_balance(user : felt) -> (res : felt):
end

@external
func register_currency{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        user : felt,
        register_amount : felt
    ):
    alloc_locals
    let (local balance) = wallet_balance.read(user)
    wallet_balance.write(user, balance + register_amount)
    return ()
end

@external
func move_currency{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        from_user : felt,
        to_user : felt,
        move_amount : felt
    ):
    alloc_locals
    let (local sender_balance) = wallet_balance.read(from_user)
    let (local receiver_balance) = wallet_balance.read(to_user)
    wallet_balance.write(to_user, receiver_balance + move_amount)
    wallet_balance.write(from_user, sender_balance - move_amount)
    return ()
end

@view
func check_wallet{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(
        user : felt
    ) -> (
        balance : felt
    ):
    alloc_locals
    let (local balance) = wallet_balance.read(user)
    return (balance)
end
```
Save as `contracts/currency.cairo`

### Compile

Compile
```
nile compile
```
Or compile this specific contract
```
nile compile contracts/currency.cairo
```

### Test

Make a new file called `test_currency.py` and populate it:

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
    contract = await starknet.deploy("contracts/currency.cairo")
    return starknet, contract

@pytest.mark.asyncio
async def test_contract(contract_factory):
    starknet, contract = contract_factory

    # Register that user 7 owns 100 units currency,
    # which might represent mainnet ether (ETH).
    await contract.register_currency(7, 100).invoke()
    # Move 2 ether from user 7 to user 789.
    await contract.move_currency(7, 789, 2).invoke()

    # Check that the currency was received.
    response = await contract.check_wallet(789).call()
    assert response.result.balance == 2
```
Run the test
```
pytest tests/test_currency.py
```

### Local Deployment

Deploy to the local devnet.
```
nile deploy currency --alias currency
```

### Interact

Write
```
nile invoke currency register_currency 7 100
```
Read
```
nile call currency check_wallet 7
```
Result: `100`

### Public deployment

Will default to the Goerli/alpha testnet until mainnet is available.
```
nile deploy currency --alias currency --network mainnet
```
Result:
```
🚀 Deploying currency
🌕 artifacts/currency.json successfully deployed to 0x04e3cd9ea4aec919c97edd7e3415e21db3cce02fda67c269e6f24e3a12f300a9
📦 Registering deployment as currency in mainnet.deployments.txt
```

Deployments can be viewed in the voyager explorer
https://voyager.online
