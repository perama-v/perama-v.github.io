---
layout: page
title:  "Merkle trees"
permalink: /cairo/background/merkle_trees/
toc: false
---

Merkle trees are used often in the Cairo ecosystem. They are useful because
they enable compact representation of data that can be attested. These features
arise because they are constructed from hashes in the shape of a tree that has
branches that divide neatly.

This binary hash tree (Merkle tree) loosely has the same shape as a round-robin graph, where
starting competitors are "leaves" and the grand final is the "root". Changing a competitor
(leaf) could the final (root). It could change the matches leading all the way up to the final
(branches).

In Merkle trees, changing the changes leaves always changes the involved branches and the root.

Below is a merkle tree implementation used in StarkWare's AMM demonstration
[tutorial](https://www.cairo-lang.org/build-a-scalable-cairo-basesd-automated-market-maker/),
which uses a transaction
[simulator](https://github.com/starkware-libs/cairo-lang/blob/master/src/demo/amm_demo/demo.py)
with accompanying
[tree helper](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/small_merkle_tree.py)
and
[Pedersen hash helper](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/crypto/starkware/crypto/signature/fast_pedersen_hash.py).
The code has been aggregated so it can be poked and explored
on its own more easily. Different system use different hash functions to build Merkle trees,
this implementation uses the Pedersen hash function.

The implementation can be used by installing the cairo-lang python library, which
contains some useful StarkWare modules:
```
from starkware.cairo.lang.vm.crypto import pedersen_hash
pedersen_hash(3, 2)
```
Result: ``576657123605396437968823113955952586959670965011232700393892413073919304299``.

(or `5.77e74`).

**Specific Example**

The Account State Root used in the AMM demo is calculated below.

The hash can be called by passing a dictionary of Accounts (an Enum) public key
and Balance (an Enum) of Token A and Token B. Equivalent to:
`{account_index: [public_key, [balanceA, balanceB]], ...}`.

```
accounts_dictionary = {
    0: Account(0x1a, Balance(100, 50)),
    1: Account(0x3a, Balance(125, 500)),
    2: Account(0xff, Balance(550, 200)),
    3: Account(0x7e, Balance(165, 40)),
    4: Account(0x33, Balance(750, 200))
}

account_tree_root = get_merkle_root(accounts_dictionary)
print(account_tree_root)
```

Which should return the Pedersen hash:
```
3262995978462033705189630496750790514901299827561453807468505774450708589253
```

Properties:
- The tree is defined as having up to 10 levels of branches, with the number of leaves doubling
at every branch level (making 2**10 leaves total).
- Each leaf value is calculated as the hash of a hash, by:
    - First taking the hash of the public_key and token A balance, `hash(public_key, A)`.
    - Taking the hash of that hash and token B balance, `hash(first_hash, B)`.
- For each leaf, the public_key and the hash form a list `[public_key, hash]`.
- Those lists are ordered from first to last (left leaf to right leaf).
`[[public_key_0, hash_0], [public_key_1, hash_2], ...]`
- The list elements are paired and hashed, forming a new list
- Those are paired and hashed, forming a new list, and so forth.
- The final hash is the root of the merkle tree.

```
import dataclasses
from fastecdsa.curve import Curve
from fastecdsa.point import Point

from starkware.crypto.signature.signature import (
    ALPHA, BETA, CONSTANT_POINTS, EC_ORDER, FIELD_PRIME,
    N_ELEMENT_BITS_HASH, SHIFT_POINT)

curve = Curve(
    'Curve0',
    FIELD_PRIME,
    ALPHA,
    BETA,
    EC_ORDER,
    *SHIFT_POINT)

LOW_PART_BITS = 248
LOW_PART_MASK = 2**248 - 1
HASH_SHIFT_POINT = Point(*SHIFT_POINT, curve=curve)
P_0 = Point(*CONSTANT_POINTS[2], curve=curve)
P_1 = Point(*CONSTANT_POINTS[2 + LOW_PART_BITS], curve=curve)
P_2 = Point(*CONSTANT_POINTS[2 + N_ELEMENT_BITS_HASH], curve=curve)
P_3 = Point(*CONSTANT_POINTS[2 + N_ELEMENT_BITS_HASH + LOW_PART_BITS],
    curve=curve)


@dataclasses.dataclass
class Balance:
    """
    Represents the balance of each of the two tokens.
    """
    a: int
    b: int


@dataclasses.dataclass
class Account:
    pub_key: int
    balance: Balance

def process_single_element(element: int, p1, p2) -> Point:
    assert element < FIELD_PRIME, 'Element integer value >= FIELD_PRIME'

    high_nibble = element >> LOW_PART_BITS
    low_part = element & LOW_PART_MASK
    return low_part * p1 + high_nibble * p2


def pedersen_hash(x: int, y: int) -> int:
    """
    Computes the Starkware version of the Pedersen hash of x and y.
    The hash is defined by:
        shift_point + x_low * P_0 + x_high * P1 + y_low * P2  + y_high * P3
    where x_low is the 248 low bits of x, x_high is the 4 high bits of x and
    similarly for y.
    shift_point, P_0, P_1, P_2, P_3 are constant points generated from the
    digits of pi.
    """
    return (HASH_SHIFT_POINT + process_single_element(x, P_0, P_1) +
            process_single_element(y, P_2, P_3)).x


def pedersen_hash_func(x: bytes, y: bytes) -> bytes:
    """
    A variant of 'pedersen_hash', where the elements and their
    resulting hash are in bytes.
    """
    assert len(x) == len(y) == 32, 'Unexpected element length.'
    return pedersen_hash(
        *(int.from_bytes(element, 'big', signed=False)
            for element in (x, y))).to_bytes(32, 'big')


class MerkleTree:
    def __init__(self, tree_height: int, default_leaf: int):
        self.tree_height = tree_height
        self.default_leaf = default_leaf

        # Compute the root of an empty tree.
        empty_tree_root = default_leaf
        for _ in range(tree_height):
            empty_tree_root = pedersen_hash(empty_tree_root, empty_tree_root)

        # A map from node indices to their values.
        self.node_values: Dict[int, int] = {1: empty_tree_root}
        # A map from node hash to its two children.
        self.preimage: Dict[int, Tuple[int, int]] = {}

    def compute_merkle_root(self, modifications: Collection[Tuple[int, int]]):
        """
        Applies the given modifications (a list of (leaf index, value)) to the
        tree and returns the Merkle root.
        """

        if len(modifications) == 0:
            return self.node_values[1]

        default_node = self.default_leaf
        indices = set()
        leaves_offset = 2 ** self.tree_height
        for index, value in modifications:
            node_index = leaves_offset + index
            self.node_values[node_index] = value
            indices.add(node_index // 2)
        for _ in range(self.tree_height):
            new_indices = set()
            while len(indices) > 0:
                index = indices.pop()
                left = self.node_values.get(2 * index, default_node)
                right = self.node_values.get(2 * index + 1, default_node)
                self.node_values[index] = node_hash = pedersen_hash(left, right)
                self.preimage[node_hash] = (left, right)
                new_indices.add(index // 2)
            default_node = pedersen_hash(default_node, default_node)
            indices = new_indices
        assert indices == {0}
        return self.node_values[1]

def get_merkle_root(accounts: Dict[int, Balance]) -> int:
    """
    Returns the merkle root given accounts state.
    accounts: the state of the accounts (the merkle tree leaves).
    """
    tree = MerkleTree(tree_height=10, default_leaf=0)
    return tree.compute_merkle_root([
        (i, pedersen_hash(
            pedersen_hash(a.pub_key, a.balance.a), a.balance.b))
        for i, a in accounts.items()])

accounts_dictionary = {
    0: Account(0x1a, Balance(100, 50)),
    1: Account(0x3a, Balance(125, 500)),
    2: Account(0xff, Balance(550, 200)),
    3: Account(0x7e, Balance(165, 40)),
    4: Account(0x33, Balance(750, 200))
}

account_tree_root = get_merkle_root(accounts_dictionary)
print(account_tree_root)

```

