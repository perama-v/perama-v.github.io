---
layout: page
title:  "Aggregated architecture"
permalink: /cairo/game/aggregated-architecture
toc: false
---

Aggregated architecture is a gaming model where players are active game designers in an
expansive community-driven open-ended game world.

In exploring the game model that is well suited to L2, I have increasingly
become convinced that the ability for players to vote is a powerful tool
that will be main stage in the coming years. Non-fungible tokens
such as the ERC-721 DOPE tokens and their unique equippable ERC-1155 Hustler characters,
empower playing rights but also are a sybil resistant too to enable 'micro-votes' (votes
that are cheap enough to make frequently).

Consider the game design for Dope Wars (the peer to peer port of the classic
calculator arbitrage game):

- Players have a Hustler, an agent equipped with unique traits, who
    - Starts with money.
    - Enters a world with NPC dealers in different locations.
    - Trade drugs with dealers to gain money.
    - Manage inventory, risk, strategy.

The game play exists as a contract that players are admitted to
and initialised based on L1 token ownership. Balances are contained
within the contract and player are traits are imported and used
to augment player characteristics based on L1 token properties.

```s
L1 -------> L2 Dope Wars
              <------
              ------>
              <------
                end
```
Consider a second module that enables players to fight for regional
power positions that extract fees from others.

```
L1 -------> L2 Dope Wars
              <------
              ------>
              <--------------> L2 combat contract
              <------
                end
```
The game is enriched by the addition of a new module that the dope
wars contract enshrines as an integrated component. Each turn
makes a call to the combat contract, and the result is used in
that turn to alter events in the Dope Wars turn.

What if there is another module that is created that could integrate
well with the game. How could that be integrated? If the Dope Wars
contract is mid-game - how do you connect the new module(s)?

The answer lies in unlocking the new L2 super-power: user voting.

De-construction of the main game to move the pan-game state to a
dedicated contract, and implementation of a governance system that
can install new components.

A temporary diversion: `The warehouse`.

## Warehouse

Imagine this as a component that is separate from the Drug Wars game logic.

I was thinking of Hustlers (players) packed into a room inside a vast
warehouse. The room has two doors and the vibrant pixel characters
are given a choice: move to room A or to room B. Players
can inspect the rooms (read the contracts) before they vote.

They vote, the votes are tallied and all the players move to winning room B.
In that room they are faced with some activity (literally anything,
a combat, a strategy game, a social challenge, a puzzle, a vending
machine that administers a beverage token to all players with health
below 50%).

All players who voted on the winning room are given some privilege
(or avoid a penalty). Other players are still brought to room B
and room A is unused. In this way, the collective is incentivised
to think about what the other voters will be choosing, and to align
their choice. But, as a whole, they must also consider the nature of the
rooms. If all players choose a room that has some negative outcome,
then all players suffer.

Negative outcomes might include:

- Loss of previously earned items.
- High risk of individual harm such as ejection from the game.
- Introduction of unfair elements to many players.
- Elements that make the game less interesting.

Having moved to room B, the players interact with the room B, and then
repeat the process. First the instantiation of the next set of rooms:

Rooms P, Q, R, S are deployed to the network, submitted for consideration
and players are able to read about the four options in directly, or
in easy to parse UIs. They perhaps discuss, criticise, rally, convince
others in discord on the merits of each room. Then they each cast a vote.
The winning room R is selected and then the contract is 'enabled for
action'. Players can then enter room R and the game continues.

The process for the development of new rooms is as follows:

- A contract submitted must conform to the Room Template.
    - Basic connector functions that admit the passage of Hustlers
    into and out of the contract.
    - Functions that can interact with core components of the Warehouse
    and game system.
    - Absence of socially prohibited features, as defined in some evolving
    document of social norms and etiquette.
- A contract may contain a small reward specifically for the deployer.
E.g., the Hustler of the developer may gain some money, or the account
of the developer could be routed some reward from each of the other players.
- The contract is deployed.
- The contract address is submitted the voting module.

## Back to modules.

Now consider the elements:

- Players
- Game logic
- Voting systems
- Assets
- Hustler traits (static L1 trait)
- Hustler RESPECT (dynamic L1 trait)
- Perishable life statistics

The system architecture could be tied together by separating individual
components so that they can be used by other components, as enabled
by votes.

```
            L2 Universal State              L2 App specific state
L1 -------> | Hustler traits    <---||---> Dope Wars
            | Hustler RESPECT   <---||---> combat module
            | ...               <---||---> warehouse base
            | Player money      <---||---> Room A/B/C/P/Q/R/S
            | Life statistics   <---||---> Gang conflict mini
            | ...               <---||---> Newer module XYZ
            | Voting module
                                           <------
                                           ------>
                                           <------
                                           Some apps may terminate.
                                           No end to the universe
```

The games all gain the ability to alter the universal state,
and the L2 voting module administers this through permission
granting from votes. The app modules may run in parallel
and may be developed and deployed asynchronously.

So in `v1`, the Dope Wars module is deployed, and rather than
saving the money for each player inside that contract, they
are saved to a single purpose contract.

```
Dope Wars: Locked architecture
    - game contract:
        - game logic
        - ids
        - money
        - drugs

Dope Wars: Modules
    - game logic contract
    - player ids contract
    - money contract
    - drugs contract

Turn
    - User calls game logic contract
        - > fetch values from four other contracts
        - > turn executed
        - > other contracts to update state
            - E.g., Player spent money to buy Krokodil, but was
            also robbed. Need to register the update to the money
            and drugs contracts.
        - > call `update_money()` on money contract.
            - money contract calls `get_caller_address()`, which
            fetches the game logic address.
            - money contract calls the governance module
            `check_permission(dope_wars_address)`. The governance
            module maintains a mapping of whether the game has
            been voted to have the power to update money.
            - If permissions granted, money is stored.
        - > call `update_drugs()` on drug contract. Perhaps
            no other games use this module, but they may later.
```

Later, the balance of the money contract can be used as the input
to other modules. Such as another game, or a reward-administering contract
to give a prize. This could also be an administrative module that is voted to
even out outlier balances that arose from the 'quirk' in a game logic.

```
| Drug Wars
| Warehouse game
| Combat game
| Gang Conflict game
| Another game
| Rewards
| Admin one-off balance rectifier
| TURF Builder
| TURF Hardware store
| ... < any other module, with vote-based 'write' capabilities >
```
The outcomes and variables for each component live in read-write
stores in a standardised fashion that can optionally be used
(read, or judiciously, write) by as-yet non-existent modules.

## Governance risk

This open architecture relies on governance being effective, 'fair'
and resistant to capture. There are many ways to approach the
structure of the voting module. There certainly exist viable
models that can be used. There is obviously a risk of governance
failure, but it seems like a healthy way for an open community
to operate.

Some ideas to mitigate failures of the governance module
could be to make app/game-specific games may be limited in their scope to affect the
universal state, or their state may exist in a temporary
cache that is only applied to the universal state after some
voting schedule that favours conservative / veto properties.
In this way, if an individual warehouse instantiation 'goes bad',
it could be prevented from messing with the main game state.
Alternatively, state-modification modules could be installed
that revert the effect of a 'bad game'.


## Governance architecture

New modules are admitted to the system in the following way:

- A contract is designed
- Deployed
- Proposal is made to the voting module
- The voters approve
- The address of the module is added to mappings
that record the `write()` permission of a certain
variable in some address

As in the warehouse module example above, the vote outcome is to
authorise the write access to the
money module. Perhaps the warehouse module requires another
variable for its game play with a clean-slate (e.g., a warehouse
specific measure of whether players are correctly selecting the
winning room each round. This new variable `oracle` points
can be stored in a new contract as a variable that other future
games can access. The oracle contract is deployed with a function
that points to the governance module for write-access checks. The
warehouse calls to update a player's `oracle` points, the oracle
store checks that the warehouse is voted/registered to write,
and the points are updated.

Future games may propose that they gain write access to `oracle`
points, bootstrapping whatever history was accrued during the
warehouse game series. Perhaps the Hustlers with many oracle
points are considered to be community members with a good sense
of community vision - or perhaps the opposite. The system allows
for new additions to use and build upon these tools in the true
DOPE-philosophy.

To restate that philosophy:

```
A shared theme, paired with ownership of unique thematic elements allows a community-driven
ecosystem to grow. The text-based token birthed a thematic universe that
allows ownership and creative participation. New participants and creators
can join and add more composable tools that can be used
in open-ended ways.
```

Here is an example of that for a particular DOPE token:

- A line of text from the original DOPE-NFT ('Uzi') can become separate ownable
token.
- It can be equipped to a new composable Hustler agent.
- It can also be manifested as a collection of pixels that represent
the physical form of an Uzi
- Pixels form the basis for static and dynamic art and graphics.
- It can also be used as a variable to influence game logic, such as a combat
mechanism or modifier.
- Usage in a game can then be used to build player
stats like health bars
- Usage can create modifiers that apply back to the weapon to represent
such as a wear-and-tear variable. An uzi might need maintenance or it loses effectiveness.
- Usage can create a new status tied to some event-based history of that object
such as a used-in-combat status. An uzi can only be traded in for rare contraband
if it has been used in a gang combat.
- An Uzi DAO subcommunity can form and work on some endeavour together.

Game logic creates opportunities to bring new variables. Rather than
having these variables be limited to just one game, they can be architected
to be open-ended and composable, like all the other elements in the DOPE universe.

The modular system also allows for contracts to be written in parallel, which opens
the possibility for pure Cairo and transpiled Solidity->Cairo contracts to coexist,
increasing mindshare opportunity.

## Concrete considerations

To action the above architecture, the following minor changes can be made to `GameEngineV1.cairo`:

- Move all stored variables that have 'composability' value (money, drugs) to
a separate contract(s).
- Make a governance contract that maintains a mapping of write() permissions.
    - Store mappings of the variable contracts as 'permitted to write'.
- Make a proxy model that can point to the primordial governance contract, but is able to
evolve when the governance/voting system architecture is designed.
- Move the combat logic to a separate player action.
    - The combat module currently exists as a subcomponent of the `GameEngineV1.cairo` contract.
    - The Drug Lord is a competitively held position within the game that attracts a 'cut' of
    trades. Autobattle combat can dethrone the local Drug Lord and appoint the victor as.
    - The Drug Lord is potentially a good 'composable' variable and can live externally, as
    are the fighters and their combat parameters.
    - The combat mechanics can be brought externally and exist as a separate action
    that a player takes outside their main game turn.
        - Player takes turn in drug wars contract. Drug lord variable is consulted externally
        to inform 'cut'/tax.
        - Player takes turn in combat contract to try to become the new Drug Lord. The Combat
        contract calls the Drug wars contract to check turn frequency. Combat game logic is
        executed and combat logic is stored in external composable contract (e.g., parameters
        of the fighter used in combat, long term damage sustained, win/loss count)
        - The next turn in the drug wars contract occurs, the winning player is now the drug
        lord for that location.
        - The locations, being limited to the 19 locations on the original DOPE NFT allow
        future game logic to be ported over to new games. See next section for more on this.

## Supporting collaboration

As more pieces in the DOPE universe come into existence, the number of useful components
that lie waiting for use increases. For example, the Drug Lord appointed through combat
as outlined in the section above can be used as an ingredient in a system that builds upon
a land-ownership model, such as the active community proposal for a TURF system.

By having the Drug Lords be constrained to the 19 locations laid out in the DOPE NFT, there
exists a very high chance that a future/tertiary component could make use both TURF and the
Drug Lord variables. For example a module on L2 could pull in the L1 turf ownership,
and the L2 Drug Lord status to administer buidable and ownable components upon the TURF plots.

Or, the Drug Lord variable could provide the basis to bootstrap a Gang system, where Drug Lords
initially have power to influence gang membership. Then as gangs mature, they have more power
and can self-regulate. The Gang system and TURF ownerships, paired with money and drug variables
populated from the gang wars `GameEngineV1` could form the basis of another system. This system
could be voted to have `write()` access to the money and drug variables, and also introduce
new composable variables that relate to gang conflict, such as weapon deterioration,
health stats, and measures of power or influence. Being in a gang with the most power might
bring benefits through rewards or privileges.

The entire system can have ballooning complexity without slowing down development. It
can enable richness without compromising the integrity
of smaller subsystems that can operate independently as interesting standalone games/elements.
In striving to make the core primitive elements composable, the scope for inclusion, creativity,
collaboration and engaging gameplay increases.

## Open possibilities

How does this tie back to current stages?

The community is able to currently engage and interact with components that are 'live'.
If you were looking to be involved by gaining ownership of the base token upon which the
entire universe spawns, how should you make your decision? What does it mean to own a
`Chain` vs an `Uzi`? What importance is the descriptive modifier of an item such as
`High on the Supply`, or the role of a `+1` for an item?

The decision in some respect is unknowable (for the universe/system may evolve indefinitely), and
in some respects it is controllable (for as a community member, you have agency to steer
the community).

The question may seem to be:

```
What trait will be the most valuable?
```

Whereas the next level up might be:

```
What new components will the community build, and how could these tie back to make
certain traits the most valuable?
```

Moving further, as the beat of the DOPE-drum rings out across the open grassy field of evolving
coordination technology, perhaps the real question is:

```
What can I bring to the community that adds value in a way that
is meaningful for myself and for others?
```

As each person follows their interests and memes value into existence, that is the ingredient
that drives value. That may be mechanism/contract/system design or it may be to create a cult
narrative that `Golf Carts` are the real `Rolls Royce`, or that `Open Toe Sandals` and
any item by `The Freelance Pharmacist` will be pivotal in an upcoming creative adventure.

Join the adventure and sculpt the future.