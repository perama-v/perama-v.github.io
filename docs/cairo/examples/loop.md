---
layout: page
title:  "Loop"
permalink: /cairo/examples/loop/
toc: false
---

Cairo supports looping through recursion.

1. The loop length is specified.
2. A looping function is called with a list of elements.
3. It checks if the element is the final one. If not, it increments and calls itself.
4. Step 3 repeats until the last element is reached.
5. The function proceeds to execute the desired operation.
6. The second last element is then reached.
7. Then the third last element, and so forth, until the first element.
8. The end of the function is reached.
9. The result is returned to the calling function.

```sh
%lang starknet

from starkware.cairo.common.alloc import alloc

@view
func read_sum() -> (sum : felt):
    let (array : felt*) = alloc()
    assert [array] = 2
    assert [array + 1] = 23
    assert [array + 2] = 1
    let (sum) = get_sum(array, 3)
    return (sum)
end

# This function has no decorator - no direct user interaction
func get_sum(array : felt*, length : felt) -> (sum : felt):
    if length == 0:
        # Start with sum=0.
        return (sum=0)
    end

    let (current_sum) = get_sum(array=array + 1, length=length - 1)
    # This part of the function is first reached when length=0.
    # The sum begins. This is the sequence: 1, 1+23 then 24+2
    let sum = [array] + current_sum
    # The return function targets the body of this function
    # 3 times before returning to the body of read_sum().
    return (sum)
end
```
Save as `loop.cairo`.

### Compile

Then, to compile:
```
starknet-compile loop.cairo \
    --output loop_compiled.json \
    --abi loop_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract loop_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x073aca6f06d6116180cbff0a5980d46fff9e0b1410ff0900a0bab762c1b5d050.
Transaction ID: 523279.
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

## Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=523279

Returns:
{
    "block_id": 25022,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/25022) and the
[contract](https://voyager.online/contract/0x73aca6f06d6116180cbff0a5980d46fff9e0b1410ff0900a0bab762c1b5d050#state)

### Interact

Then, to have the contract loop over the array and compute the sum:

```
starknet call \
    --network=alpha \
    --address 0x73aca6f06d6116180cbff0a5980d46fff9e0b1410ff0900a0bab762c1b5d050 \
    --abi loop_contract_abi.json \
    --function read_sum

Returns:
26
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.