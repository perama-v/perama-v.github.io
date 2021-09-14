---
layout: page
title:  "Market Maker"
permalink: /cairo/game/market-maker/
toc: false
---


An constant product market maker (automated market maker) that accepts a market inventory and a
user trade, and returns the updated market and user balances. The constant product swap formula
enforces that the product of two inventory items (item `a` and `b`) are the same pre- and
post-trade. This automatically increases prices for scarce items and lowers prices of
plentiful items.

`market_a_post * market_b_post = market_a_pre * market_b_pre`

A user gives `a` and receives `b`. The purpose of the formula is to find how much `b` the user
receives.

```py
market_a_post = market_a_pre + user_gives_a  # User gives.
market_b_post = market_b_pre - user_gets_b  # User takes.

# Substitue into original
(market_a_pre + user_gives_a) * (market_b_pre - user_gets_b) =  market_a_pre * market_b_pre

# Expand, note that the right hand term is cancelled. Simplify:
user_gets_b = market_b_pre * user_gives_a / (market_a_pre + user_gives_a)
```

Building a stripped-down version of this
[contract](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/starknet/apps/amm_sample/amm_sample.cairo).

```sh
%lang starknet
%builtins range_check

from starkware.cairo.common.math import assert_nn_le, unsigned_div_rem

# The maximum value an item can have.
const BALANCE_UPPER_BOUND = 2 ** 64

# Accepts an AMM state and an order, instantiates AMM, swaps, returns balances.
# The market gains item `a` loses item `b`, the user loses item `a` gains item `b`.
@external
func trade{range_check_ptr}(
    market_a_pre : felt, market_b_pre : felt, user_gives_a : felt) -> (
    market_a_post : felt, market_b_post : felt, user_gets_b : felt):
    # Prevent values exceeding max.
    assert_nn_le(market_a_pre, BALANCE_UPPER_BOUND - 1)
    assert_nn_le(market_b_pre, BALANCE_UPPER_BOUND - 1)
    assert_nn_le(user_gives_a, BALANCE_UPPER_BOUND - 1)

    # Calculated how much item `b` the user gets.
    # user_gets_b = market_b_pre * user_gives_a // (market_a_pre + user_gives_a)
    let (user_gets_b, _) = unsigned_div_rem(
        market_b_pre * user_gives_a, market_a_pre + user_gives_a)

    # Calculate what the market is left with.
    let market_a_post = market_a_pre + user_gives_a
    let market_b_post = market_b_pre - user_gets_b

    # Ensure that all value updates are >= 1.
    assert_nn_le(1, user_gives_a)
    assert_nn_le(1, user_gets_b)
    assert_nn_le(1, market_a_post - market_a_pre)
    assert_nn_le(1, market_b_pre - market_b_post)

    # Check not items conjured into existence.
    assert (market_a_pre + market_b_pre + user_gives_a) = (
        market_a_post + market_b_post + user_gets_b)
    return (market_a_post, market_b_post, user_gets_b)
end

```
Save as `MarketMaker.cairo`.

### Compile

Then, to compile:
```
starknet-compile MarketMaker.cairo \
    --output MarketMaker_compiled.json \
    --abi MarketMaker_contract_abi.json
```

### Test

Make a new file called `MarketMaker_contract_test.py` and populate it:

```py
import os
import pytest

from starkware.starknet.compiler.compile import (
    compile_starknet_files)
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

# The path to the contract source code.
CONTRACT_FILE = os.path.join(
    os.path.dirname(__file__), "MarketMaker.cairo")


# The testing library uses python's asyncio. So the following
# decorator and the ``async`` keyword are needed.
@pytest.mark.asyncio
async def test_record_items():
    # Compile the contract.
    contract_definition = compile_starknet_files(
        [CONTRACT_FILE], debug_info=True)

    # Create a new Starknet class that simulates the StarkNet
    # system.
    starknet = await Starknet.empty()

    # Deploy the contract.
    contract_address = await starknet.deploy(
        contract_definition=contract_definition)
    contract = StarknetContract(
        starknet=starknet,
        abi=contract_definition.abi,
        contract_address=contract_address,
    )

    market_a_pre = 300
    market_b_pre = 500
    user_a_pre = 40  # User gives 40.
    res = await contract.trade(market_a_pre, market_b_pre, user_a_pre).invoke()
    (market_a_post, market_b_post, user_b_post, ) = res

    assert market_a_post == market_a_pre + user_a_pre
    assert market_b_post == market_b_pre - user_b_post
```

The packages required for testing are installed by default with cairo-lang.
Run the test:

`pytest MarketMaker_contract_test.py`.

Check that the tests passed:
```
=== 1 passed, 1 warning in 11.26s ===
```

### Deploy

Then, to deploy:
```
starknet deploy --contract MarketMaker_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x07f9ad51033cd6107ad7d70d01c3b0ba2dda3331163a45b6b7f1a2952dac0880
Transaction ID: 133739
```

### Monitor

Check the status of the transaction:
```
starknet tx_status --network=alpha --id=133739

Returns:
{
    "block_id": 14520,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/14520) and the
[contract](https://voyager.online/contract/0x07f9ad51033cd6107ad7d70d01c3b0ba2dda3331163a45b6b7f1a2952dac0880)

### Interact

Trades aren't stored in this contract so we can use `call` (view) instead of `invoke` (write)

Create a trade for a market with 500 A, 1000 B and a user who is giving 30 A.

```
starknet call \
    --network=alpha \
    --address 0x07f9ad51033cd6107ad7d70d01c3b0ba2dda3331163a45b6b7f1a2952dac0880 \
    --abi MarketMaker_contract_abi.json \
    --function trade \
    --inputs 500 1000 30

Returns:
530 944 56
```
The market gained 30 A and the user gained 56 B.

Now try with the pairs reversed.
Create a trade for a market with 1000 A, 500 B and a user who is giving 30 A.

```
starknet call \
    --network=alpha \
    --address 0x07f9ad51033cd6107ad7d70d01c3b0ba2dda3331163a45b6b7f1a2952dac0880 \
    --abi MarketMaker_contract_abi.json \
    --function trade \
    --inputs 1000 500 30

Returns:
1030 486 14
```
The market gained 30 A and the user gained 14 B. The user was giving an item that the market
has plenty of, and you can see that they received a smaller quantity than in the first example.

This contract may now be called by the GameEngineV1.cairo contract.