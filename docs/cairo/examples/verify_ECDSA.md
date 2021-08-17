---
layout: page
title:  "Verify an ECDSA Signature"
permalink: /cairo/examples/verify_ecdsa/
toc: false
---

Cairo has a builtin that allows an ECDSA signature to be verified. For example, a user registers
their public key, then authorises a transfer of some number of coins. The contract verifies that
the contract contains a message signed properly by the authorised person.

Steps:
1. Make a message to sign. E.g., the number "13579", which might represent a quantity.
2. Hash the message using the Pedersen hash function.
3. Obtain a private key. E.g., Generate a random one as shown below.
4. Sign the message hash using the private key.
5. Record the two parts of the signature, sig_r and sig_s.

Generate a random private key. With the Cairo python package installed run the following
code as a `.py` file, or type `python3` in the terminal and paste the following:
```py
from starkware.crypto.signature.signature import (sign,
    get_random_private_key, private_to_stark_key, pedersen_hash)

message = 13579
message_hash = pedersen_hash(message)
private_key = get_random_private_key()
signature = sign(msg_hash=message_hash, priv_key=private_key)
r = signature[0]
s = signature[1]
public_key = private_to_stark_key(private_key)


print(f"""
The pedersen hash of message {message} is:
{message_hash}
was signed by the public key:
{public_key}

The signature_r is:
{r}
The signature_s is:
{s}
""")
```
Result:
```sh
The pedersen hash of message 13579 is:
2255487090060981340412778270523682108366421523848318719210969003889439916982
was signed by the public key:
336651705928807190204460653884405974785205047669862626875647782176669707088

The signature_r is:
1893532933103991730127061797833157421284043895315515223059211080812570729772
The signature_s is:
2767847065606260480373088228849935790434610797527549053315487023265903912514
```

The function `verify_ecdsa_signature()` from the Cairo common library
accepts the four numbers above as arguments:

- message_hash
- public_key
- signature_r
- signature_s

The curve is different from the one used on Ethereum, so a signed message by an L1 wallet cannot
directly be checked inside an L2 contract as of Cairo v0.3.0.

### Contract

Below is the contract that will check the signed message.

```sh
%lang starknet
%builtins ecdsa

from starkware.cairo.common.signature import verify_ecdsa_signature
from starkware.cairo.common.cairo_builtins import SignatureBuiltin

# ecdsa: a builtin that tracks the signature in the Cairo trace.
# SignatureBuiltin: a struct used to represent the signature.
# ecdsa_ptr: a pointer (*) to the signature struct.
# The pointer is passed implicitly '{}'.
@view
func check_signature{ecdsa_ptr: SignatureBuiltin*}(
        message, public_key, sig_r, sig_s) -> ():
    # If the signature is incorrect, this will fail.
    verify_ecdsa_signature(message_hash, public_key, sig_r, sig_s)
    return ()
end
```
Save as `verify_ecdsa.cairo`.

### Compile

Then, to compile:
```
starknet-compile verify_ecdsa.cairo \
    --output verify_ecdsa_compiled.json \
    --abi verify_ecdsa_contract_abi.json
```
### Deploy

Then, to deploy:
```
starknet deploy --contract verify_ecdsa_compiled.json \
    --network=alpha

Returns:
Deploy transaction was sent.
Contract address: 0x061ebf39a3dbdc61d2881ba9d9820d92174b84495c4aea0d41dc545086eceb29
Transaction ID: 152698
```

*Note:* Remove the zero after the `x`, 0x[0]12345. E.g., 0x0123abc becomes 0x123abc.

### Monitor

Check the status of the transaction:

```
starknet tx_status --network=alpha --id=152698

Returns:
{
    "block_id": 34065,
    "tx_status": "PENDING"
}
```
The [block](https://voyager.online/block/34065) and the
[contract](https://voyager.online/contract/0x61ebf39a3dbdc61d2881ba9d9820d92174b84495c4aea0d41dc545086eceb29#state)

### Interact

Then, to interact, call the function using the arguments from above (in same order,
message_hash, public_key, sig_r, sig_s). This call would fail if the signature did not pass
the checks enforced by the contract.

```
starknet call \
    --network=alpha \
    --address 0x61ebf39a3dbdc61d2881ba9d9820d92174b84495c4aea0d41dc545086eceb29 \
    --abi verify_ecdsa_contract_abi.json \
    --function check_signature \
    --inputs \
    2255487090060981340412778270523682108366421523848318719210969003889439916982 \
    336651705928807190204460653884405974785205047669862626875647782176669707088 \
    1893532933103991730127061797833157421284043895315515223059211080812570729772 \
    2767847065606260480373088228849935790434610797527549053315487023265903912514
```

Status options:

- NOT_RECEIVED: The transaction has not been received yet (i.e., not written to storage).
- RECEIVED: The transaction was received by the operator.
    - PENDING: The transaction passed the validation and is waiting to be sent on-chain.
        - REJECTED: The transaction failed validation and thus was skipped.
        - ACCEPTED_ONCHAIN: The transaction was accepted on-chain.


Visit the [voyager explorer](https://voyager.online/) to see the transactions.