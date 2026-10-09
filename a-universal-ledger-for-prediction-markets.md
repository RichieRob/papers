# A universal ledger for prediction markets

7 Oct 2026

## The question

Paradigm's Opportunity Markets describes a record label that funds prediction markets on its own signings so that scouts are paid for finding the artists it will want. The label asks which artists it will sign this year, up to ten. Anyone can add an artist, all year. The label puts its own money behind the markets, concentrated where it wants it, between roughly 1% and 30%, and no more than it chooses per artist. Every token has to be fully backed, so a scout holding a signed artist is paid a dollar at year end whatever else happened. Privacy during the scouting window is a separate question, set aside here.

How would you build that?

## Separating solvency and liquidity

You could build it as a set of bespoke contracts. Write the label's liquidity rule into the market, fix the year, fix how many can be signed, and it would work for that label, that year and that rule. The next sponsor, with a different rule, a different horizon and a different list, would need another set.

The more general answer is to separate solvency from liquidity. Solvency is whether every token is backed, and it has to hold for everyone, always, whatever anyone traded. Liquidity is somebody standing ready to buy and sell at a price with their own money, and it is a matter of choice: who, how much, in what band, for how long. If one contract does solvency and nothing else, then liquidity can be supplied any way at all on top of it. The label can run a private market maker with its own rules. Anyone else can run a public one. An automated market maker and an order book can quote the same tokens side by side. None of them can put a scout's dollar at risk, because the contract underneath holds every account to the same backing, and none of them needs to be designed again when the next sponsor wants something different.

The label's rule, the year, the size of the list and the number signed are contingencies. The thing underneath should not know about them.

## The landscape today

Gnosis's conditional token framework is the origin of the separation. A dollar mints a full set of outcome tokens for a question, the framework holds the dollar, and the tokens trade wherever anyone will quote them. It backs and does not quote.

The separation works, but the framework has architectural limits, and what is built on top of it has compromised around them. Without a full analysis, where it lands is a series of isolated yes or no markets, one per answer, with trading through a central limit order book.

That shape has three drawbacks.

- Liquidity only unifies on an order book. Without one, liquidity sits in separate pots, one per binary, each locked to its own market and idle while the others trade. An order book lets one maker with one inventory quote across every market at once, which is why a CLOB ends up being essential.
- The venue is centralised. Resting orders are matched by an operator, so prices are made at one point of control, with implications for privacy as well as openness.
- A new market starts empty. It has no price until somebody chooses to quote one, so the moment a market is most useful, when a scout has just found an artist, is the moment it has no liquidity.

## Current implementations

Two things follow that nobody can build in today's landscape, however they design the market on top.

- A market with several sources of liquidity at once: two different automated market makers, say, each quoting the same tokens from its own pool.
- An automated market maker and an order book quoting the same market side by side.

The point is that there is no universal solvency engine for prediction markets. There is Gnosis CTF, which is innovative but is not quite up to the role. What is needed is one trusted contract that guarantees solvency for any question with full backing, designed for multiple different liquidity sources so any market can simply be put on it and liquidity can come from wherever it comes, order books and AMMs.

## Our ledger

We have built the universal solvency engine for prediction markets. One contract holds every account, checks before each trade that it can cover its worst case, and does nothing else.

What it enables:

- Any number of liquidity sources on one market: two AMMs, an AMM beside an order book, a sponsor's private book, each just an account held to the same backing.
- A market maker that deposits what it is prepared to lose and quotes however it likes.
- A market with any number of outcomes that can grow while it trades: a new outcome is added to a live market with its own token, no new pool and no new collateral, and every token is an ERC20 that trades anywhere.
- A new market with a price on every outcome from the moment it opens, from one deposit.

## The first implementation

Our first implementation is [TopFlight.fun](https://football.topflight.fun), a perpetual prediction market on the Premier League.
