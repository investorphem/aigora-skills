### Contact
Telegram @investorphem

### CELO payout wallet
0xec24bafbc989a9be5f6f0ead8848753b5e4ae0b6

### Aigora profile URL
https://aigora.org/services/42220_0x8004a169fb4a3325136eb29fa0ceb6d2e539a432_9760

### Surface
Agent profile page

### Network
Mainnet — Celo (chainId `42220`)

### What happened?
I went to hire a listed agent and could not. I opened an agent whose services carry a price and
looked for the way to pay it — there is no payment section anywhere on the profile. I then looked
for the way to message the agent, since this repo's README advertises paid DMs — there is no DM
section either.

The README describes Aigora as:

> the Celo agent marketplace: ERC-8004 identity + reputation, discoverable agent profiles,
> **x402-paid DMs, and bounty escrow**.

Of those four, identity and profiles are real and work. I could not find a surface for the other
two. Prices are displayed on service rows, but a price with no pay action is a label, not an offer.

**Why this is the finding underneath several others already filed.**

Two open PRs approach this from either side without naming it:

- **#8** reports that a service row shows "this service costs 0.01 USDC" without declaring the
  HTTP method or payment protocol, so a buyer cannot tell *how* to pay. It treats paying as
  something the buyer does off-platform, against the endpoint directly.
- **#4** asks for a "Try this agent" dry-run and states plainly that "there is no way at all to
  validate a listing from the buyer's perspective" — and its proposal explicitly stops short of
  settling, offering to "simulate the flow **without settling a payment**".

Both are correct and neither says the quiet part: **there is no transaction layer to fix.** #8 is
asking for better metadata so a buyer can go transact somewhere else; #4 is asking for a simulation
because real settlement is not on the table. The catalog can describe commerce but cannot host it.

**Why it matters more than a missing feature.**

1. **A marketplace that cannot take payment is a directory.** The value proposition over a plain
   ERC-8004 registry scan is that discovery leads to a transaction. Today discovery leads to a URL
   you must go integrate against yourself — which you could have got from the registry.

2. **It explains why reputation is unearnable.** I filed separately that Aigora has no way to give
   feedback and that a natively-registered agent shows zero clients on-chain. These are the same
   hole seen from two ends: reputation should be a by-product of transactions, and there are no
   transactions. Ship payments and honest reputation follows almost for free — every settled
   payment is a counterparty entitled to rate the agent. Without payments, the only feedback that
   can ever exist is unattested, which is exactly the weakness visible in the current scores.

3. **The x402 rails already exist on Celo.** This hackathon is producing many x402-priced agents,
   and Celo runs a facilitator at `api.x402.celo.org`. An agent listing already carries a price, a
   token and an endpoint. The distance between "displays a price" and "settles a payment" is
   smaller here than in most marketplaces, which is what makes the gap surprising.

### Steps to reproduce
1. Open https://aigora.org on Celo mainnet with a wallet connected.
2. Open any agent whose Services list shows a non-zero price.
3. Look for any control to pay for or purchase that service. There is none — the price is display
   text.
4. Look for any control to message or DM the agent, as the README's "x402-paid DMs" implies. There
   is none.
5. Compare against the README's description of the product, quoted above.

### Logs / console output
_No response_

### Transaction / agent ID
Observing account `0xec24bafbc989a9be5f6f0ead8848753b5e4ae0b6`, owner of agent `9760`
(registered through Aigora on Celo mainnet).

### Anything else
Suggestions, cheapest first:

1. **Say what is live.** If paid DMs and bounty escrow are roadmap rather than shipped, the README
   should distinguish them from identity and profiles. I went looking for features because the
   product's own description says they exist; a "coming soon" marker costs nothing and saves that
   trip.
2. **Make the price actionable for x402 services.** For a service already marked x402, the profile
   could fetch the endpoint's 402 challenge and render the real payment requirements — price,
   token, network, payTo — straight from the source. That also fixes #8's complaint that listed
   prices are self-reported and unverifiable, using data the endpoint already returns.
3. **Then settle through the Celo facilitator.** Once the challenge is displayed, a "Pay and call"
   action is the natural next step, and it is the point at which Aigora becomes a marketplace
   rather than a catalog.
4. **Use settled payments as the reputation gate.** A counterparty that actually paid is the one
   whose rating means something. This closes the loop with the reputation issue rather than
   patching it separately.

Context: I build a payments agent that settles x402 on Celo mainnet, so I came to Aigora as a
buyer as much as a lister — the question I wanted answered was "can I find an agent here and pay
it?" That is the one thing I could not do.
