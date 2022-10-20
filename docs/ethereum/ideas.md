---
layout: page
title:  "Open ideas"
permalink: /ethereum/ideas
toc: true
---

A system to increase the rate at which ideas
are made available to society.

Contents

- [Introduction](#introduction)
- [Background](#background)
- [Design](#design)
- [Future applications](#future)
- [Organic Manifestations](#organic-manifestations)

```
[---Open~Idea---]

(*) Open your own ideas to the world (*)
```

# Introduction

**Premise:** Anything that humans create must first be
an idea. An increase in the rate of ideation might translate
to an increase in the rate useful goods produced. This might
result in a faster time-to-completion for any number of goals
humans have as a collective.

**Terminal Goal:**

- Solve problems faster.

Intermediate goals:

- Faster production of tools.
- Faster tool ideation.
- Lower idea propagation friction.
- Simple idea publication system.
  - Participation incentive for economically rational actors.
  A retrospective content-address-attribution royalty model.
  - Fast in simple-case, defers conflict resolution. Append-only public
  graph of ideas where it is simple for an ideator to "do the right thing".
  - Co-exist with existing idea systems (e.g., patent). Is opt-in, adopted
  because it is a better experience for the user (ideator).

```
TL;DR
A timestamped list of address -> idea_hash mappings that make it
easy to voluntarily pay people whose ideas led you to generate wealth.
```

# Background

## Idea origination

Ideas here are imagined as compositions of previous ideas. Knowledge
of a stick and a rock enables one to imagine how a stick plus a rock
makes a hammer. The idea is born, then the hammer is made.

The process for how an individual pieces together ideas is complex.
In general, the ability to compose abstract idea-blocks together
seems to be a good model for the qualia of the act. Have a look at
the diagrams in this paper and imagine how someone might have
has an idea for a bow and arrow. How do the ideas held in the mind
come to form new combinations? Consider that a laser finder on a bow
and arrow is a reasonable idea once a laser is a building block (bow + laser).
The idea is a bad idea for the bowmaker if a laser has not already been
invented.

Self-directing robotic arrows are perhaps an idea that is unreasonable now,
but perhaps not forever (arrow + microrobot).

Lombard, Marlize & Haidle, Miriam. (2012). Thinking a Bow-and-Arrow Set: Cognitive Implications of Middle Stone Age Bow and Stone-Tipped Arrow Technology. Cambridge Archaeological Journal. 22. 237-264. 10.1017/S095977431200025X.

## Ideation rate an cumulative idea count.

The ideation rate is likely to be some function of the

- Number of people trying to generate ideas. The system does not address
this factor.
- The number of ideas from which to draw from and combine
in unique ways. Ideas can be drawn from only when they coexist in a
single mind. Thus, an idea is most useful when it is present in the minds
of all the ideators.

Humans generate ideas in opaque ways. Some notable historical figures
generate vast quantities of excellent ideas, others fewer. The subject of
this work is to instead look at the population-level of ideation. A
pool of humans each generating ideas at some rate that is a function of their
nature and circumstance.

**Individual idea pool:** A person might be imagined as having a reserve of
'potential ideas' that they may convert into 'real ideas'. A person mentally
occupied (e.g., busy with some work or task), may not
generate any ideas from this pool. Another individual who intentionally
tries to generate ideas might produce a large proportion of this pool.

One potential model for ideation is combinatorial sampling. The mind might
draw elements from the pool and combine them, evaluate the result and then
explore the idea if it passess this filter. If the rate of sampling
is constant, then the rate of 'good' ideas is a function of the merit of
the idea in achieving some arbitrary (and possible poorly-defined) goal.
If the idea is composed of a collection of real and effective tools
that are available for use, then the idea is more likely to seem good (E.g.,
an idea that relies on a known realistic/manufacturable alloy is better than one that uses a non-existent alloy). Therefore, if the ideator has a large pool of contemporary
ideas from which to sample from (E.g., they are aware of the idea of the alloy),
they might have a higher rate of 'good' ideas.

**Practical abstraction:** A primary idea, one what is based on available technologies, can serve as the basis for a secondary idea. The primary idea is not
yet in-production, but can be readily imagined as an available technology. The
secondary idea is a 'good' idea, because in the mind of the ideator, who is aware
of the primary idea, the result is within achievable limits using current technology.
E.g., If the primary idea is to make a tool from an alloy, secondary idea is to use the tool to build some other thing.

**Pool growth:** An increase in the total number of ideas available for consumption
increases the rate of ideation at the individual level, which then increases
the rate of new ideas produced by and for society. If it is true that ideas are
combinations of existing ideas, then an
individual exposed to more ideas will have a greater idea-count potential.
The possible number of new ideas available for the individual to is likely a
superlinear function of available ideas. As the number of ideas available for an
individual to draw from and combine grows, the number of potential combinations
grows superlinearly.

For every new idea published and distributed, the pool of ideas in each ideator
grows. There is possibly some relationship where the growth rate of new ideas
is proportional to the number of distributed ideas, which would cause
exponential growth.

## Historical context

As we develop technologies to distribute ideas, the pace of innovation
increases. Many domains have benefited from the ability to instantly distribute ideas to a global audience. The audience may receive, interact and contribute to ideas
collectively and within short periods of time. Collaboration is certainly
an evolving power that society will benefit from moving forward.

However, there is a clear delineation where some ideas are "not for distribution".
By publishing an idea you could lose control of it. Yet by keeping it private, you
miss out on collaboration and dissemination for use. The solution: delay the time-to-publication, by first following some procedure of protection such as
creating a patent, creates friction that slows global progress. A patent might
take years to be granted. What of the loss of collaboration at the time of ideation,
when the idea is fresh and exciting?

What happens to the rate of ideation globally if time-to-publication goes from
years to days? The quantity of actionable ideas in that future world
could be vastly higher than what we currently expect.


# Design

A timestamped list of `address -> idea_hash` mappings and a separate
graph of content and weighted attributions.

The blockchain provides:

- A persistent destination for payments to be routed. Payments can be made
retrospectively years later.
- A timestamp. The idea could be falsely attributed to a another person.


## Topology

The blockchain-based graph component consists of a chain of hashes stored in a smart contract.

```
H1 <- H2 <- H3 <- H4
```
The hashes are used to fetch data from a content-addressable peer to peer
filestore. Fetching the latest file returns structured data which
contains the parent hash, which enables the entire graph to be traversed.

Each file consists of data used to construct an attribution graph.
An idea may attribute to prior nodes in the graph.

## Timestamp

The hash of the root of the system is posted to a blockchain regularly.
A client calls the blockchain to query the latest hash. Then fetches the
data from the storage network.

A new user may update the hash by submitting a state transition that
appends content to the graph. They submit the root hash and their new
hash, which is stored on the blockchain.

## Data availability

Stored in layer 2 contract:

- Current latest idea hash as storage in

Stored in peer to peer filestore:

- Sequence of idea_hashes
- Ideas, consisting of
  - Author L2 public key address
  - Idea description
  - Attributions

Why not put the attributions as storage on L2 - the system is opt-in,
and it is reasonable to assume that anyone making a payment will
parse the graph correctly when gathering the addresses and
payment proportions to make a donation.

## Idea data structure

Fields that can be used by a client to construct the content and
attribution graph.

```
{
  "author_public_key": 0x12345,
  "idea": "idea x",
  "attributions": [
    (3ea..8c, 0.3),
    (34b..d7, 0.6),
    (581..ae, 0.1)
  ]
}
```

## Payments

Someone wishing to distribute funds to the graph uses the client
to generate a list of public keys and weights. The list is then passed
to the smart contract along with the funds. The payment is split
and distributed by weight.

The client generates a transaction payload that can be submitted to a
payment contract to divide the funds.

## Attributions

Attributions are pairs of hashes and weights. An idea may list multiple
attributions, where the weights sum to 1. A payment is routed to the attributed
nodes with these weights. E.g., `[(3ea..8c, 0.3), (34b..d7, 0.6), (581..ae, 0.1)]`

Payments decay naturally along the graph. A project raises 10 million
dollars of funding and had 10% pledged to on idea hash, called
"main idea". Thus they use
the client to generate the list of payments. That idea attributed 40%
to other ideas, who in turn attributed various percentages to other ideas.

```
- 0.1 main idea (1kk)
  - 0.6 Send: 600k
  - 0.4 Attributions (400k)
    - 0.7 Secondary idea 1 (280k)
      - 0.5 Send: 140k
      - 0.5 Attributions (140k)
        - 0.9 Tertiary idea 1 (126k)
          - 0.6 Send: 75.6k
          - 0.4 Attributions (50.4k)
            - 0.8 Quaternery idea 1 (40.4k)
            - 0.2 Quaternery idea 1 (10.8k)
        - 0.1 Tertiary idea 2 (14k)
    - 0.2 Secondary idea 2 (80k)
      - 0.9 Send: 72k
      - 0.1 Attributions (8k)
        - 0.7 Tertiary idea 3 (5.6k)
        - 0.3 Tertiary idea 4 (2.4k)
    - 0.05 Secondary idea 3 (20k)
      - 0.6 Send: 12k
      - 0.4 Attributions: (8k)
        - 0.9 Tertiary idea 5 (7.2k)
        - 0.1 Tertiary idea 6 (0.8k)
    - 0.05 Secondary idea 4 (20k)
      - 0.1 Send: 2k
      - 0.9 Attributions (18k)
        - 0.6 Tertiary idea 7 (10.8k)
        - 0.4 Tertiary idea 8 (7.2k)
```
Without listing the entire next layer (quaternery ideas), you can see how
the graph distributes funds according to the weights that the ideators
declared at time of publication.

The ideator could under-allocate the percentage that they choose
to attribute to preceding ideas. However, given that the graph
is public, there is a degree of social pressure that disincentivises
this. The ideator hopes to attract funds, and therefore is likely
to be honest about the contribution (the nature of which can be seen).

There could be a requirement to attribute at least 10% to other ideas,
or a more aggressive proportion (60%), to create very broad distribution
that extends many levels deep.

### Key management

Ideators can append a new public key to the graph by signing with
their old key. A client traverses the graph and updates latest
keys when determining a payment addresses.

Lost keys are not addressed here.

### Intellectual property

The ideas presented in the graph are not a claim of intellectual property.
They are a presentation of an idea as-is, for use by anyone for any purpose,
without warranty. Should two parties want to use the same idea, they
are able to - and must compete against each other to provide the most compelling
product.

Should the idea already exist as a published patent or copywright article
elsewhere that predates the published idea, the idea can be viewed with this
in mind. It is not knowable whether the ideator was aware of this. Anyone
looking to make a product based on this idea should perform basic checks to
determine if there is reasonable evidence that the idea is the intellectual
property of some other person, as defined by a system external to this
system.

The system is a voluntary opt-in royalty attribution system to encourage
the transmission and development of new ideas. An equivalent way of thinking
of the system is a way to publish with a permissive license with an inbuilt
pro-social tipjar, organised in a convenient directory.

### Minor edits

An idea can be improved upon in small ways by making a new contribution,
and attributing a high (99%) proportion of the attributions to the main
idea. In this way, edits are encouraged. Undesirable edits may be ignored,
and anyone can choose any part of the graph to direct funds towards.
Thus if there is low-value addition that seems to be an attempt to
extract value from an existing good idea, then this can be assessed
and navigated easily.

The client could have standard rates, ranging from "minor edit" (99%),
"Interesting idea" (70%), "Well-researched novel product" (30%) to
"Standalone genius invention" (10%). This might encourage people
to select more pro-social attribution percentages, which benefits
the system as a whole.


## Journey of the Ideator

Let us examine the psychology of the person who creates an idea.

**Moment of inspiration**: In the moment of ideation, the mind is subsumed by euphoria.
Ideas are inherently joyful things to create, and it is a natural response to
want to share an idea. This forms the basis for the system. Users
ultimately want to see their idea propagate to others and the system enables
that.

**Extraction**: Having real world responsibilities and goals,
the ideator tempers their excitement and starts to map the idea to their
needs. Perhaps the idea could form the basis for a new business that
provides personal wealth? The system provides a mechanism for personal value
accrual, proportional to the value that the idea generates for the world.

**Protection**: Perhaps by keeping the idea private they
can better achieve this, protecting against other actors taking the opportunity.
The ideator knows that the world respects the concept of origination, and
that the first person to document the idea properly has the favour
of the legal system. The system allows ideas to be timestamped, providing
a reassurance that they will be recognised as being first.

**Scribing**: What must be done to secure an idea? Perhaps there
is some degree of documentation in some particular format. Perhaps
the idea must not be shared until the documentation is complete and
secured in a proper fashion. Perhaps secured in one particular country?
The system provides a simple mechanism
for ideas to be documented in a standard format.

**Attribution**: The idea feels new, but it may be inspired by something
similar. The ideator might have a vague sense of a collection of ideas
that led to this particular idea. They know that an invention is not
something done in isolation, and is likely to feel gratitude for
these prior works. It might be a natural desire to recognise these publicly
at some point, acknowledging their role in this idea. The system provides
a mechanism to add attribution connections to prior ideas.

**Building**: The idea is a good one, perhaps good enough to start
thinking about bringing it into existence. This might start
a journey in a direction that this person is not suited to - perhaps
into creating a business, dealing with investors, or regulation.
It is a non-trivial consideration to build out an idea. The system provides
a mechanism for ideas to be found by people suited to taking ideas
to product.


## Ideator-Builder separation

While it makes sense for some ideas to be built by the person who thinks
of them, it perhaps is not always the case. Some people are dreamers,
distracted in their world of creative imagination. It may be
efficient instead to match builders to ideas. People who are laser
focused and effective at managing practical considerations.

Anyone looking to make a product can examine the graph and start building.
No permission needed, only a commitment to return some profit to the
source of the graph.


## Publication makes you the subject of attributions

If you publish your idea, there is a chance that someone will see that idea
and later recognise that it had a role in their idea.

## Stranger-funding

If a person gets funding to build an idea from the graph, there could
be a social norm where a percentage is immediately directed to the
idea graph. Thus, if you had an idea and the idea is so good that
someone was able to raise money, then you are rewarded.

This also creates an incentive for people to broadcast their idea. The
more people that are aware of the idea, the higher the chance that
someone will adopt the idea and raise funds for production of that idea (or
a derivative idea).

## Idea refinement

Anyone who sees a way to improve a published idea can publish that improvement
and receive attribution. They are now a stakeholder in this local area of the
idea-graph and are also incentivised to propogate the idea to others.

## Decay-by-depth Royaltes

Connections become very small after some depth.

This prevents ideas early in the system from becoming huge nodes, accruing
wealth beyond their relatively distant contribution.

## Benefit from your competitor

An idea that is published and built out by the same person might have
another person try to build the same idea. If the second person
succeeds, they fund the idea, thus rewarding the ideator. While this
could undercut some business proposition of the first company, from the
perspective of the ideator, this is: 1) The idea becoming successfully
deployed in the real world, 2) Personal wealth accrual. Thus, when
considering whether to publish the idea, this scenario is appealing.


# Future

The ideas graph is a substrate that is a source of ideas that can be
used by anyone for any purpose. It is a place where extremely profitable
ideas could appear, and there could exist separate systems to curate
and develop these ideas into products.

### Idea translation

Individuals, groups, companies and DAOs can rapidly organised around
published ideas and begin work. Perhaps they could hire the original
ideator, whose address may be sent messages to initiate coordination.

A DAO may dedicate itself to specifications: Building out concise
specs or prototypes of ideas from the graph - and selling them on to
other companies.
With retention of the original attribution instructions. Thus
'walking' the idea further toward completion, and directing some
funds to the ideator in the process.

DAOs may be deployed with any number of structures, including membership
based systems that rewards different contributors, or
tokenised incentive structures for investors to finance. Good ideas
can be funded very quickly, with responsibility to deliver a product
being separated from the ideator, instead placed on the entity making
some promise or claim about a future product.

DAOs could issue membership NFTs, useful for directing investment funds
in different ways to build out the idea. The community could grow around
the idea and the shared vision, knowing that if successful, there is
a social contract to donate some proportion of funds from the
"builder community" back to the "ideas community".

### Prediction and Curation markets

There are perhaps novel mechanisms that could allow market participants
to take actions around the quality and probability of ideas, teams,
and implementations. For example, a prediction market could arise
based a prototype deployment, or on whether
VC funding will be sourced before a certain date.

## General remarks

This is a loose and simple framework. There are more complex
tools that could be integrated, but there is perhaps a benefit to starting
simple.

One feeling I have is that the core reasons ideas are slow to emerge is
that people are afraid. By providing a simple and neutral mechanism that
is easy to understand, people might be less afraid to release their
ideas. There is likely a strong desire for people to share their ideas
and have them be the subject to the internet-swarm, whose members
can contribute to the idea in complementary ways.



This idea is an:
```
[---Open~Idea---]

(*) Open your own ideas to the world (*)
```



# Organic Manifestations

In recongnition that a de novo knowledge graph would be an enormous undertaking, it may be wise to consider where the implementation might natrually arise.

Consider GitHub as a graph of knowledge where there is already
a system for 'attestation of personal contribution'. Imagine if you
could use a piece of open source software and click a donate button
that automatically routed payments to all the developers who made commits.

One could even create a donations client that walks the links of a github project, fetching the contributors involving in the linked projects.

It could be a cultural norm in the future to perform retroactive public goods funding by this mechanism - e.g., if you used/forked a piece of free and open source software, you could willingly and publicly direct some funds along to the humans that were involved in
commiting the code that you ultimatley used.

One important distinction is that such a practice would ideally be
seen as a voluntary act, and that using FOSS without donating is also reasonable. I could see a future where it is compelling at the individual level (personal fulfilment) and at the organisational level (social engagement) to participate in retroactive funding in this way.

### The inception of the database

Consider a database with mappings of github users to ECDSA public keys.

A system could work as follows:

1. Donator inputs the name of a repository they like.
2. Software pulls the github users that have made commits.
3. Users are looked up in the user->ECDSA key database.
4. List of public keys are returned.
5. A multi-send contract on an ECDSA-compatible L2 is passed a donation amount and the list of addresses.
6. Funds are routed.
7. Perhaps some social certificate is optionally minted as a fun way to generate attention or clout. Perhaps owners of these tokens could be regarded as good-actors that other systems could use when looking
for list of 'community-minded' people.

Being tightly coupled to the Ethereum community, a good testbed
could be Ethereum-related projects. There could be a site that
allows users to sign in with their github account and attest to a public key.

An individual is incentivised to volunteer an address if they have ever made a commit to a public repo that might attract donations.

Perhaps the implementation is just a public repo with some simple rules
allowing only addresses to be appended to a large list. The commit
author for a given address is enforced by github authentification.


