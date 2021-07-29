---
layout: page
title:  "Read and write"
permalink: /cairo/examples/read_write/
toc: false
---

To write to a state variable, you need to send a transaction. This is stored in StarkNet
and is included as rollup data on Ethereum.

To read a state variable, no transaction is needed.

```sh
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.storage import Storage

@storage_var
func stored_felt() -> (res : felt):
end

# Function to get the stored number.
@view
func get{storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}() -> (res : felt):
    let (res) = stored_felt.read()
    return (res)
end

# Function to update the stored a number (field element).
@external
func save{
        storage_ptr : Storage*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(input : felt):
    stored_felt.write(input)
    return ()
end

```
Save as `read_write.cairo`.

### Compile

Then, to compile:
```
starknet-compile read_write.cairo \
    --output read_write_compiled.json \
    --abi read_write_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract read_write_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x033402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9.
Transaction ID: 497976.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=497976

Returns:
{
    "block_id": 23896,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/23896) and the
[contract](https://voyager.online/contract/0x33402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9#state)

### Interact

Then, to interact, invoke the save function with any number. This requires a transaction:

```
starknet invoke \
    --network=alpha \
    --address 0x33402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9 \
    --abi read_write_contract_abi.json \
    --function save \
    --inputs 10000000000

Returns:
Invoke transaction was sent.
Contract address: 0x033402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9.
Transaction ID: 497977.
```
Read the stored number using a call to the get() function. This does not required a
transaction:
```
starknet call \
    --network=alpha \
    --address 0x33402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9 \
    --abi read_write_contract_abi.json \
    --function get

Returns:
10000000000
```

All field elements are from a finite field defined by the specific prime number:

`prime = 2**251 + 17 * 2**192 + 1`

Which:

- In binary is a 252 digit number (~1.00 * 2**251): `100000000000000000000000000000000000000000000000000000010001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001`
- In decimal is a 76 digit number (~3.62 * 10**75): `3618502788666131213697322783095070105623107215331596699973092056135872020481`
- In hexadecimal is a 65 digit number (~8.00 * 16**64): `0x800000000000011000000000000000000000000000000000000000000000001`

Everything in Cairo is represented in decimal form. Storing the a number of equal size to the
prime number causes failure:

```
starknet invoke \
    --network=alpha \
    --address 0x33402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9 \
    --abi read_write_contract_abi.json \
    --function save \
    --inputs 3618502788666131213697322783095070105623107215331596699973092056135872020481

Returns:
Error: StarkException: (500, {'code': <StarknetErrorCode.OUT_OF_RANGE_ENTRY_POINT_SELECTOR: 12>, 'message': 'Call data element 3618502788666131213697322783095070105623107215331596699973092056135872020481 is out of range'})
```

Storing the prime number minus 1 suceeds:
```
starknet invoke \
    --network=alpha \
    --address 0x33402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9 \
    --abi read_write_contract_abi.json \
    --function save \
    --inputs 3618502788666131213697322783095070105623107215331596699973092056135872020480

Returns:
Invoke transaction was sent.
Contract address: 0x033402a2a76ef75b36eec4689f601961a5b23b30ea4d3f5e83b4f095002823a9.
Transaction ID: 497979.
```

Trying with other values yields these results:

- Valid
    - save(0). Zero allowed.
    - save(1). Numbers 1 onward allowed.
    - save(prime - 1). This is the largest allowed number.
- Not valid
    - save(prime). No using the prime number of the field.
    - save(prime + 1). No overflow beyond the prime.
    - save(-1). No negative numbers.
    - save(-(prime) - 1). No wrap around below from negative prime.
    - save(0x20). No hexadecimal numbers where field elements are expected.
    - save(0b1101). No binary numbers where field elements are expected.

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.