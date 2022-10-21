---
layout: page
title:  "DopeWarsV1"
permalink: /cairo/game/game_engine_v1/
toc: false
---

The `v1` game engine is a lean MVP. It predominantly consists of mappings stored
in StarkNet state, an AMM that executes trades, and some probabilistic events.
The main game engine is a contract that stores the mappings and calls out to
other contracts for player information and AMM trades.

The StarkNet contract engine will consist of:

```
builtins
imports
mappings
functions
```

Each value the contract is represented by a `felt` (field element), the native type used in
Cairo programs.

In Cairo maps are created with storage functions with one or more parameter fields.
Nested maps are created by adding more parameters:

- `func map_a(key : felt) -> (value : felt)`.
    - Equivalent to  `map_a[key]`.
- `func map_b(key1 : felt, key2) -> (value : felt)`.
    - Equivalent to `map_b[key1][key2]`.

Below is the game engine, with main function `have_turn()`, which examines state,
receives instructions, calls another contract to execute trade logic, then update state.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage
from starkware.cairo.common.math import assert_nn_le


#### Other Contract Info ####
# Address of previously deployed MarketMaker.cairo contract.
const MARKET_MAKER_ADDRESS = 0x07f9ad51033cd6107ad7d70d01c3b0ba2dda3331163a45b6b7f1a2952dac0880
# Modifiable address pytest deployments.
@storage_var
func market_maker_address() -> (address : felt):
end

# Declare the interface with which to call the Market Maker contract.
@contract_interface
namespace IMarketMaker:
    func trade(market_a_pre : felt, market_b_pre : felt,
        user_gives_a : felt) -> (market_a_post : felt,
        market_b_post : felt, user_gets_b : felt):
    end
end

#### Game key ####
# World: 40 locations (city, suburb) pairs.
# city_ids: [1,10]
# suburb_ids: [1,4]
# user_ids: [1,10000]
# item_ids: [1,11]. 0=money, 1=item1, 2=item2, ..., etc.
# buy=0, sell=1.

#### Game state ####
# Specify user, item, return quantity.
@storage_var
func user_has_item(user_id : felt, item_id) -> (count : felt):
end

# Location of users. Input user, retrieve city.
@storage_var
func user_in_city(user_id : felt) -> (city_id : felt):
end

# Location of users. Input user, retrieve suburb.
@storage_var
func user_in_suburb(user_id : felt) -> (surburb_id : felt):
end

# Returns the count of some item for a given market, defined by location.
@storage_var
func market_has_item(city_id : felt, suburb_id : felt,
    item_id : felt) -> (count : felt):
end

# A market has their money in discrete accounts, one account per item.
@storage_var
func market_has_money(city_id : felt, suburb_id : felt,
    item_id : felt) -> (count : felt):
end


#### Admin Functions for Testing ####
# Sets the address of the deployed MarketMaker.cairo contract.
@external
func set_market_maker_address{storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*, range_check_ptr}(
        address : felt):
    # Used for testing. This can be ca constant on deployment.
    market_maker_address.write(address)
    return ()
end


# Creates an item-money market pair with specific liquidity.
@external
func admin_set_market_amount{storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*, range_check_ptr}(city_id : felt,
        suburb_id : felt, item_id : felt, item_quantity : felt,
        money_quantity : felt):
    # Set the quantity for a particular item in a specific market.
    # E.g., item_id 3, a market has 500 units, 3200 money liquidity.
    market_has_item.write(city_id, suburb_id, item_id, item_quantity)
    market_has_money.write(city_id, suburb_id, item_id, money_quantity)
    return ()
end


# Creates an item-money market pair with specific liquidity.
@external
func admin_set_user_amount{storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*, range_check_ptr}(user_id : felt,
        item_id : felt, item_quantity : felt):
    # Set the quantity for a particular item for a given user.
    # E.g., item_id 3, a user has market has 500 units. (id 0 is money).
    user_has_item.write(user_id, item_id, item_quantity)
    return ()
end


#### Game functions ####
# Actions turn (move user, execute trade).
@external
func have_turn{syscall_ptr : felt*, storage_ptr : Storage*,
        pedersen_ptr : HashBuiltin*, range_check_ptr}(user_id : felt,
        city_id : felt, suburb_id : felt, buy_or_sell : felt,
        item_id : felt, amount_to_give : felt):
    # E.g., Sell 300 units of item. amount_to_give = 300.
    # E.g., Buy using 120 units of money. amount_to_give = 120.
    alloc_locals
    assert_nn_le(buy_or_sell, 1)  # Only 0 or 1 valid.
    # Move user
    user_in_city.write(user_id, city_id)
    user_in_suburb.write(user_id, suburb_id)

    # giving_id = 0 if buying, giving_id = item_id if selling.
    local giving_id = item_id * buy_or_sell
    # receiving_id = item_id if buying, receiving_id = 0 if selling.
    local receiving_id = item_id * (1 - buy_or_sell)

    # A is always being given by user. B is always received by user.

    # Pre-giving balance.
    let (local user_a_pre) = user_has_item.read(user_id, giving_id)
    assert_nn_le(amount_to_give, user_a_pre)
    # Post-giving balance.
    local user_a_post = user_a_pre - amount_to_give
    # Save reduced balance to state.
    user_has_item.write(user_id, giving_id, user_a_post)

    # Pre-receiving balance.
    let (local user_b_pre) = user_has_item.read(user_id, receiving_id)
    # Post-receiving balance depends on MarketMaker.

    # Record pre-trade market balances.
    if buy_or_sell == 0:
        # Buying. A money, B item.
        let (market_a_pre_temp) = market_has_money.read(
            city_id, suburb_id, item_id)
        let (market_b_pre_temp) = market_has_item.read(
            city_id, suburb_id, item_id)
    else:
        # Selling. A item, B money.
        let (market_a_pre_temp) = market_has_item.read(
            city_id, suburb_id, item_id)
        let (market_b_pre_temp) = market_has_money.read(
            city_id, suburb_id, item_id)
    end
    # Finalise values after IF-ELSE section (handles fp).
    local market_a_pre = market_a_pre_temp
    local market_b_pre = market_b_pre_temp

    let market_maker = MARKET_MAKER_ADDRESS
    # ***Uncomment below for pytest for dynamic market address***
    # let (market_maker) = market_maker_address.read()

    # Execute trade by calling the market maker contract.
    let (market_a_post, market_b_post,
        user_gets_b) = IMarketMaker.trade(market_maker,
        market_a_pre, market_b_pre, amount_to_give)

    # Post-receiving balance depends on user_gets_b.
    let user_b_post = user_b_pre + user_gets_b
    # Save increased balance to state.
    user_has_item.write(user_id, receiving_id, user_b_post)

    # Update post-trade states (market & user, items a & b).
    if buy_or_sell == 0:
        # User bought item. A is money.
        market_has_money.write(city_id, suburb_id,
            item_id, market_a_post)
        # B is item.
        market_has_item.write(city_id, suburb_id, item_id,
            market_b_post)
    else:
        # User sold item. A is item.
        market_has_item.write(city_id, suburb_id, item_id,
            market_a_post)
        # B is money.
        market_has_money.write(city_id, suburb_id, item_id,
            market_b_post)
    end
    return ()
end


#### Read-Only Functions for Testing ####
# A read-only function to inspect user state.
@view
func check_user_state{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(user_id : felt) -> (money : felt,
        id1 : felt, id2 : felt, id3 : felt, id4 : felt, id5 : felt,
        id6 : felt, id7 : felt, id8 : felt, id9 : felt, id10 : felt,
        city : felt, suburb : felt):
    alloc_locals
    # Get the quantity held for each item.
    let (local money) = user_has_item.read(user_id, 0)
    let (local id1) = user_has_item.read(user_id, 1)
    let (local id2) = user_has_item.read(user_id, 2)
    let (local id3) = user_has_item.read(user_id, 3)
    let (local id4) = user_has_item.read(user_id, 4)
    let (local id5) = user_has_item.read(user_id, 5)
    let (local id6) = user_has_item.read(user_id, 6)
    let (local id7) = user_has_item.read(user_id, 7)
    let (local id8) = user_has_item.read(user_id, 8)
    let (local id9) = user_has_item.read(user_id, 9)
    let (local id10) = user_has_item.read(user_id, 10)
    # Get location
    let (local city) = user_in_city.read(user_id)
    let (local suburb) = user_in_suburb.read(user_id)
    return (money, id1, id2, id3, id4, id5, id6, id7, id8, id9, id10,
        city, suburb)
end


# A read-only function to inspect pair state of a particular market.
@view
func check_market_state{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(city : felt, suburb : felt,
        item_id : felt) -> (item_quantity : felt, money_quantity):
    alloc_locals
    # Get the quantity held for each item for item-money pair
    let (local item_quantity) = market_has_item.read(city, suburb,
        item_id)
    let (local money_quantity) = market_has_money.read(city, suburb,
        item_id)
    return (item_quantity, money_quantity)
end

```
Save as `GameEngineV1.cairo`.

### Compile

Then, to compile:
```
starknet-compile GameEngineV1.cairo \
    --output GameEngineV1_compiled.json \
    --abi GameEngineV1_contract_abi.json
```

### Test

Make a new file called `GameEngineV1_contract_test.py` and populate it:

```py
import os
import pytest

from starkware.starknet.compiler.compile import (
    compile_starknet_files)
from starkware.starknet.testing.starknet import Starknet
from starkware.starknet.testing.contract import StarknetContract

# The path to the contract source code.
ENGINE_CONTRACT_FILE = os.path.join(
    os.path.dirname(__file__), "GameEngineV1.cairo")
MARKET_CONTRACT_FILE = os.path.join(
    os.path.dirname(__file__), "MarketMaker.cairo")

# The testing library uses python's asyncio. So the following
# decorator and the ``async`` keyword are needed.
@pytest.mark.asyncio
async def test_record_items():
    # Compile the contracts.
    engine_contract_definition = compile_starknet_files(
        [ENGINE_CONTRACT_FILE], debug_info=True)
    market_contract_definition = compile_starknet_files(
        [MARKET_CONTRACT_FILE], debug_info=True)

    # Create a new Starknet class that simulates the StarkNet
    # system.
    starknet = await Starknet.empty()

    # Deploy the contracts.
    market_contract_address = await starknet.deploy(
        contract_definition=market_contract_definition)
    engine_contract_address = await starknet.deploy(
        contract_definition=engine_contract_definition)

    # Create contract Objects to interact with.
    market_contract = StarknetContract(
        starknet=starknet,
        abi=market_contract_definition.abi,
        contract_address=market_contract_address,
    )
    engine_contract = StarknetContract(
        starknet=starknet,
        abi=engine_contract_definition.abi,
        contract_address=engine_contract_address,
    )

    # Save the market address in the engine contract so it can call
    # the market maker contract.
    await engine_contract.set_market_maker_address(
        address=market_contract_address).invoke()

    # Set up a scenario. A user who will go to some market and trade
    # in some item in exchange for money.
    user_id = 3
    item_id = 5
    city_id = 9
    suburb_id = 4
    # User has small amount of money, but lots of the item they are selling.
    user_money_pre = 300
    user_item_pre = 55
    # Market has lots of money, not a lot of the item it is receiving.
    market_item_pre = 10
    market_money_pre = 12000
    # Set action (buy=0, sell=1)
    buy_or_sell = 1  # Selling
    # How much is the user giving (either money or item)
    item_quantity = 20  # 20 of the item.

    # Create the market.
    await engine_contract.admin_set_market_amount(city_id, suburb_id,
        item_id, market_item_pre, market_money_pre).invoke()
    # Give the user item.
    await engine_contract.admin_set_user_amount(user_id, item_id,
        user_item_pre).invoke()
    # Give the user money (id=0).
    await engine_contract.admin_set_user_amount(user_id, 0,
        user_money_pre).invoke()
    pre_trade_user = await engine_contract.check_user_state(
        user_id).invoke()
    print('pre_trade_user', pre_trade_user)
    pre_trade_market = await engine_contract.check_market_state(
        city_id, suburb_id, item_id).invoke()
    print('pre_trade_market', pre_trade_market)

    # Execute a game turn.
    await engine_contract.have_turn(user_id, city_id, suburb_id,
        buy_or_sell, item_id, item_quantity).invoke()

    # Inspect post-trade state
    post_trade_user = await engine_contract.check_user_state(
        user_id).invoke()
    print('post_trade_user', post_trade_user)
    post_trade_market = await engine_contract.check_market_state(
        city_id, suburb_id, item_id).invoke()
    print('post_trade_market', post_trade_market)

    # Check money made. (Assert user money (index 0) post > money pre)
    assert post_trade_user[0] > user_money_pre
    # Check gave items. (Assert user item quantity (index=item_id) post <> money pre)
    assert post_trade_user[item_id] < user_item_pre
    # Check city is set
    assert post_trade_user[11] == city_id
    # Check suburb is set
    assert post_trade_user[12] == suburb_id

    # Check that the market gained item
    assert post_trade_market[0] > market_item_pre
    # Check that the market lost money
    assert post_trade_market[1] < market_money_pre

    # Check nothing was minted
    units_pre = user_money_pre + user_item_pre + market_item_pre + \
        market_money_pre
    units_post = post_trade_user[0] + post_trade_user[item_id] \
        + post_trade_market[0] + post_trade_market[1]
    assert units_pre == units_post
```

If using pytest, search for 'pytest' in the contract and
uncomment the line with the notice. It will allow for dynamic
deployment address for the `MarketMaker.cairo` contract.

The packages required for testing are installed by default with cairo-lang.
Run the test:

`pytest GameEngineV1_contract_test.py`.

Check that the tests passed:
```
=== 1 passed, 1 warning in 32.11s ===
```

### Deploy

Then, to deploy:
```
starknet deploy --contract GameEngineV1_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a
Transaction ID: 133827
```

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=133827

Returns:
{
    "block_id": 14530,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/14530) and the
[contract](https://voyager.online/contract/0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a)

### Interact

This contract is for testing and has no game state, to add state, use the admin functions.

Let use create the following scenario: A user finds a market willing to pay a lot for
an item they have a lot of. They chose to go there and sell all they have.

```py
user_id = 733
item_id = 3  # The item to be traded.
city_id = 7
suburb_id = 2
# User has small amount of money, but lots of the item they are selling.
user_money_pre = 30
user_item_pre = 200
# Market has lots of money, not a lot of the item it is receiving.
market_item_pre = 10
market_money_pre = 12000
# Set action (buy=0, sell=1)
buy_or_sell = 1  # Selling
# How much is the user giving (either money or item)
amount_to_give = 200
```

First add `200` of item `3` to user `733` with
`admin_set_user_amount(user_id, item_id, item_quantity)`.

```
starknet invoke \
    --network=alpha \
    --address 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a \
    --abi GameEngineV1_contract_abi.json \
    --function admin_set_user_amount \
    --inputs 733 3 200

Returns:
Invoke transaction was sent.
Contract address: 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a
Transaction ID: 133902
```

Now use the same function with `item_id=0` to add `30` money to the user.

```
starknet invoke \
    --network=alpha \
    --address 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a \
    --abi GameEngineV1_contract_abi.json \
    --function admin_set_user_amount \
    --inputs 733 0 30

Returns:
Invoke transaction was sent.
Contract address: 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a
Transaction ID: 133919
```

Now add money to a market in city `7`, suburb `2` using
`admin_set_market_amount(city_id, suburb_id, item_id, item_quantity, money_quantity)`

```
starknet invoke \
    --network=alpha \
    --address 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a \
    --abi GameEngineV1_contract_abi.json \
    --function admin_set_market_amount \
    --inputs 7 2 3 10 12000

Returns:
Invoke transaction was sent.
Contract address: 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a
Transaction ID: 133936
```

Now it is time to test the core logic of the game. Imagining that all the locations
and other players were initialised. This player sees the good prices at the dealer
we have created and choses to travel there and make the trade by selling `200` of item `3`.
The player (or GUI) would use the function:
`have_turn(user_id, suburb_id, buy_or_sell, item_id, amount_to_give)`

```
starknet invoke \
    --network=alpha \
    --address 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a \
    --abi GameEngineV1_contract_abi.json \
    --function have_turn \
    --inputs 733 7 2 1 3 200

Returns:
Invoke transaction was sent.
Contract address: 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a
Transaction ID: 133978
```
The state of the user can then be examined with a call function to
`check_user_state(user_id)`

```
starknet call \
    --network=alpha \
    --address 0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a \
    --abi GameEngineV1_contract_abi.json \
    --function check_user_state \
    --inputs 733

Returns:
11458 0 0 0 0 0 0 0 0 0 0 7 2
```
The user has sold all their item `3` and now has `11458` money.

Play around yourself by sending transactions to this contract (`0x02c91`)
with the CLI or
[with the explorer](https://voyager.online/contract/0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a#writeContract)

Feel free to use the admin controls to create scenarios and see how the mechanics work.


