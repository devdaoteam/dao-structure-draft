# DevDao structure

Structure in 3 types of smart contracts (counting NFT and collection as
one contract):
- Main DevDao Contract (Tokensig)
- Distributor for employees of the "service sector" DevDao
- Distributor-project, or Project Contract


# Main DevDao Contract

> work in progress, only calculation part finished

### Briefly:

This is a wallet from which messages can be sent only by the decision of
the token vote.

At the beginning of voting, he receives the issue of this token and
calculates the maximum votes. Each member has 1 vote by default. The
number of his tokens adds to him a certain number of votes, which depends
on the number of Dao participants.

Voting takes place by sending tokens to the contract. The contract counts
the vote for each member of the Dao and blocks the tokens.

Upon reaching the majority of votes, the message is sent and
their tokens are returned to everyone. If the required number of votes is
not collected, after a few days (let's say three), tokens are returned to
everyone and nothing is sent.

### Calculation of votes

> To prevent a situation where an active member of the Dao owns by the
> majority of tokens and can win the vote alone, weight the number of
> tokens in the voting is not constant and is calculated from
 the number of
> Dao members.

Let $E$ be the emission of tokens at the start of voting. $M$ - the number
of Dao participants.
\
One of the Dao members sends his vote and the
number of tokens $C$.

$P$ is its share of the entire emission of $E$.

$P = \frac{C}{E}$

Denote by $V$ the number of votes that the participant's share gives $P$
and introduce the influence factor (coefficient) $F$, then:

$F = \frac{V}{P}$

The more participants there are, the more difficult it is to get hold of all the tokens.
Therefore, the larger the Dao, the more votes tokens can mean. $F \sim M$.

$F = rM$, where $r$ is the proportionality coefficient.

##### To output the coefficient, let's simulate the situation:

Let's say if the Dao consists of $M_{1}=2$ people, owning half of the tokens
( $C = \frac{1}{2}\times{E} \Rightarrow P = 0.5$ ) should give $V_{1} = 1$
voice. Then $F_{1} = \frac{V_{1}}{P_{1}} = \frac{1}{0.5} = 2$.

And even if the Dao consists of $M_{2}= 102$ people, owning
half of the tokens ( $P = 0.5$ ) will give $V_{2} = 11$ votes. Then
$F_{2} = \frac{V_{2}}{P_{2}} = \frac{11}{0.5} = 22$.

##### Output $r$:

$r = \frac{\Delta F}{\Delta M} = \frac{F_{2} - F{1}}{M_{2} - M_{1}} = \frac{22 - 2}{102 - 2} = \frac{20}{100} = 0.2$


### Application 

##### Formula for calculating the maximum number of votes $A$

$A = M + rM$ or $A = M(r + 1)$

##### The formula for calculating the votes of $K$ for each voter

$K = 1 + P \times F$ or $K = 1 + \frac{C}{E}\times rM$

##### Expressions for integration into code

`max_votes = total_members * (r + 1)`

`one_user_votes = 1 + (user_tokens / emission) * r * total_members`


# Project Contract

DevDao is divided into many projects.

The project is a contract with the NFT collection interface.

Its NFTs belong to those who were engaged in development or promotion
the project, i.e. I did some work for him.

The number of NFTs in the project does not exceed 250.

The project is minted by one transaction from the DevDao Tokensig.

In the initializing transaction, the addresses and shares of all
project participants are indicated at once. Participants can be **only members of DevDao**
who participate in voting in the DevDao Tokensiga. During
inita are minted immediately and all NFT for participants.

There are 2 important dicts in the contract:
1. Stores information about shares by the NFT index as a key, example:
    - `0, 0.1` - owner of NFT with index 0 gets 10%
    - `1, 0.5` - next 50%
    - `2, 0.2` - 20%
    - `3, 0.1` - 10%
    - `777, 0.1` - And this is a balance record for the DevDao fund

2. Stores balances in a similar way.
    - `0, 15*10^9` - The owner of an NFT with an index of 0 has 15 TON on the balance
    - `1, 75*10^9` - This one has 75
    - `2, 0` - This one apparently recently withdrew money
    - `3, 15*10^9` - This one has 15
    - `777, 10*10^9` - And this is a balance record for the DevDao fund

The contract of the project receives all the money earned by the product that was
developed within the framework of this project. The contract charges money to the balances during
the second dictionary according to the shares in the first. If
there is more than 50 TON of money on the fund's balance sheet, it sends 90% of this balance to the DevDao Token
and 10% to the distributor to intangible employees.

NFT can also accept a request from its owner to withdraw funds from
the project contract. Then the NFT generates and sends a message to the collection with the desired op
and with its index and owner's address in the body. The project contract checks
by the index that the message is really from the NFT and outputs its
balance to the owner.


# Distributor for DevDao service sector

There are guys who do not develop products, but help them with an idea, or
advice, or design. Remuneration for such people is carried out through
this contract.

According to the structure of withdrawal, balance and shares, it is
identical to the project contract. With the exception, perhaps, of 10% in
the fund. Here everything is distributed only between the owners of the
NFT.

The distributor can be completely controlled by the main Dao contract.
"Tokensig" can change shares, screw up a new NFT, transfer an existing
one.
