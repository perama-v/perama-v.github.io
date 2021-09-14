---
layout: page
title:  "User Registry"
permalink: /cairo/game/user-registry/
toc: false
---

This contract accepts serves as a database of users
from which rounds of `DopeWarsV{x}.cairo` can be initialized.

If the mechanism is a message bridge, the contract might look like
this [contract](https://www.cairo-lang.org/docs/_static/l1l2.cairo).

If the mechanism is Merkle proof-based, the design will be different.
