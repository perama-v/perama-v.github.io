---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_3
toc: false
---


# Part III - Specifying

- Hypothesis: The address-appearance-index can be described without prototype-specific
or network-specific details.
- Methods: Create a specification for the index using ssz, making decisions and parameters explicit.
- Conclusion: Specification can be described, allowing for alternate implementations and networks.

(Continued from [poking part 2](part_2.md))

## Some Origins

Consider that if the index is useful, a recipe for integrating it would be good.
How should the recipe be formed?

With what `kolla` should the `prōtos` page be adhered with?

## Some Scope

Now I am thinking about what the assumptions about the world that the index spec makes.
Additionally, wwhat are the assumptions that someone can make about the index.

What does a good specification have? I try not to spend too much time on this.
For the idea of this index itself may be poorly formulated, and a robust specification
will be a waste.

## Some Mimicry

I see that the python spec has been working well for the [beacon chain](#beacon-chain) and the
[portal network](#portal-network), and like the format.

Perhaps the target audience for this new address-appearance-index-spec will be familiar
with this format?

## Some Clarity

The process of writing out the spec helps to decide what are parameters of the system,
and how they are decided. E.g., What is the maximum possible length of a list of items
that just keeps growing? How many bytes should your network name occupy? Strings
are abolished. There are no vagaries here.

I write down the spec in python and then represent each on as a struct in rust, seasoning
with some ssz-derivers. I don't go as far as making the spec executable. The process
writing a script to extract the python looks to be a chore, and so I leave it open
for a few bugs.

Transactions:
```python
class AppearanceTx(Container):
    block: uint32
    index: uint32
```
List of transactions:
```python
class AddressAppearances(Container):
    address: ByteVector[DEFAULT_BYTES_PER_ADDRESS]
    appearances: List[AppearanceTx, MAX_TXS_PER_VOLUME]
```
List of lists of transactions:
```python
class AddressIndexVolumeChapter(Container):
    address_prefix: ByteVector[NUM_COMMON_BYTES]
    identifier: VolumeIdentifier
    addresses: List[AddressAppearances, MAX_ADDRESSES_PER_VOLUME]
```
List of lists of lists of transactions:
```python
class AddressChapter(Container):
    identifier: ChapterIdentifier
    members: List[AddressIndexVolumeChapter, MAX_VOLUMES]
```
**Vector** of lists of lists of lists of transactions:
```python
class AddressAppearanceIndex(Container):
    chapters: Vector[AddressChapter, NUM_CHAPTERS]
```

There it is. The final container. It seems silly that you can lay it out like this,
and have the final data be so compact.

I group the data by Chapters (address groups) rather than Volumes (time-based groups)
because that is what the user wants. So it is more natural to have a directory or database
for each Chapter than each Volume.

Continue on to [poking part 4](part_4.md)

----
## References

### Portal Network

Lightweight protocol access by resource constrained devices.

https://github.com/ethereum/portal-network-specs/blob/master/README.md

### Beacon chain

The specification for Phase 0 -- The Beacon Chain.

https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md
