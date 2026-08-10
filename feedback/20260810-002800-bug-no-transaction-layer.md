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
I went to hire a listed agent and could not. I opened an agent whose service carries a price and
looked for the way to pay it — there is no payment control anywhere on the profile. I then looked
for the way to message the agent, since this repo's README advertises paid DMs — there is no DM
surface either.

To be precise about scope: this is about **Aigora's own product surface**, not about Celo or x402.
The rails exist and work — Celo runs an x402 facilitator at `api.x402.celo.org`, my own agent
settles x402 payments on Celo mainnet, and the agent I tried publishes complete, machine-readable
payment requirements (shown below). The gap is that **Aigora provides no in-product way to pay a
listed service or send a paid DM**; a buyer must leave the marketplace and integrate against the
endpoint themselves.

The README describes Aigora as:

> the Celo agent marketplace: ERC-8004 identity + reputation, discoverable agent profiles,
> **x402-paid DMs, and bounty escrow**.

Identity and profiles are real and work. I could not find an in-product surface for paid DMs.
Prices are displayed on service rows, but a price with no pay action is a label, not an offer.

**The data a checkout would need is already public.** Agent `9339` (AjoAI) lists a service at
$0.05. Fetching that service's own endpoint returns everything a payment flow requires — method,
protocol, network, amount, token and `payTo`:

```shell
$ curl -s https://ajo-ai-tan.vercel.app/api/invoke
{"name":"AjoAI underwriting",
 "description":"AjoAI savings-credit underwriting report for a Celo wallet",
 "method":"POST","protocol":"x402","network":"eip155:42220",
 "price":{"amount":"0.05","currency":"USDC"},
 "payTo":"0x8974881E39a5eF62214929B6CaA6EC0C6e7D47c7",
 "available":true, ... }
```

Aigora renders the `$0.05` and nothing else. This is not a missing-data problem.

**Why this sits underneath several reports already filed.**

Three open PRs approach it from different sides without naming it:

- **#8** reports that a service row shows "this service costs 0.01 USDC" without declaring the HTTP
  method or payment protocol, so a buyer cannot tell *how* to pay. It treats paying as something
  the buyer does off-platform, against the endpoint directly.
- **#20** reports that `x402Support` reads false even when every service carries a USDC price — the
  listing metadata does not reflect the payment capability the agent actually has.
- **#4** asks for a "Try this agent" dry-run, states that "there is no way at all to validate a
  listing from the buyer's perspective", and explicitly stops short of settling, offering to
  "simulate the flow **without settling a payment**".

All three are correct. The common cause is that Aigora describes commerce but does not host it: #8
wants better metadata so a buyer can transact elsewhere, #20 is about a capability flag that cannot
be acted on in-product, and #4 proposes a simulation because real settlement is not available to
propose.

**Why it matters more than a missing feature.**

1. **A catalog that cannot take payment is a directory.** The value over a plain ERC-8004 registry
   scan is that discovery leads to a transaction. Today discovery leads to a URL you must go
   integrate against yourself — which the registry could have given you.

2. **It explains why reputation is unearnable.** I filed separately (#34) that Aigora exposes no way
   to give feedback, and that my natively-registered agent `9760` returns **0 clients** from the
   Reputation Registry's `getClients()`. These are the same hole from two ends: reputation should be
   a by-product of transactions, and there are no in-product transactions, so the only feedback that
   can exist today is unattested.

3. **The distance to close it is unusually short.** The listing carries a price, a token and an
   endpoint; the endpoint publishes the full payment requirements; Celo already runs the
   facilitator. "Displays a price" to "settles a payment" is a smaller gap here than in most
   marketplaces.

### Steps to reproduce
Observed on Celo mainnet, 2026-08-10 ~01:00 UTC, wallet connected, desktop Chrome.

1. Open the profile of agent **AjoAI** (`#9339`, registered 2026-06-13, owner `0x8974…47c7`):
   `https://aigora.org/services/42220_0x8004a169fb4a3325136eb29fa0ceb6d2e539a432_9339`
2. In its Services list (6 rows), locate the first row — type `web` / `rest`, priced **$0.05**,
   endpoint `https://ajo-ai-tan.vercel.app/api/invoke`, described as "Savings-credit underwriting
   report for a wallet … Paid in USDC over x402 (Celo facilitator)". The other five rows
   (`metrics`, `A2A`, `email`, `MCP`, `DID`) are free.
3. Look for any control to pay for or purchase that service — a buy/pay/hire button, a checkout, a
   payment panel. There is none; the `$0.05` renders as static text.
4. Look for any control to message or DM the agent, as the README's "x402-paid DMs" implies. There
   is none.
5. Confirm the payment requirements are nonetheless published by that endpoint:
   `curl -s https://ajo-ai-tan.vercel.app/api/invoke` — output above.

One caveat for completeness: `POST`ing the same URL at the time of writing returned
`503 {"error":"Payment facilitator unavailable; try again shortly."}` from the agent's own
integration, so I could not capture the 402 challenge itself. That is the agent's transient state,
not Aigora's — the `GET` descriptor above is the verified artifact, and it is what a checkout would
read.

### Logs / console output
```shell
$ curl -s -o /dev/null -w "%{http_code} %{content_type}\n" https://ajo-ai-tan.vercel.app/api/invoke
200 application/json

$ curl -s -X POST https://ajo-ai-tan.vercel.app/api/invoke \
    -H 'Content-Type: application/json' -d '{"address":"0xec24…e0b6"}'
HTTP 503
{"error":"Payment facilitator unavailable; try again shortly."}
```

### Observed account / agent ID
No transaction was attempted or sent — **the finding is the absence of any control that would
initiate one**, so there is no transaction hash to cite, and that absence is the evidence.

Observing account `0xec24bafbc989a9be5f6f0ead8848753b5e4ae0b6`, owner of agent `9760` (registered
through Aigora on Celo mainnet). Agent observed: **AjoAI `9339`**, Celo mainnet.

### Anything else
Suggestions, cheapest first.

**1. Say what is live.** If paid DMs and bounty escrow are roadmap rather than shipped, the README
should distinguish them from identity and profiles. I went looking because the product's own
description says they exist; a "coming soon" marker costs nothing and saves that trip.

**2. Make the price verifiable for x402 services.** For a service already marked x402, display the
payment requirements read from the endpoint itself — price, token, network, `payTo` — rather than
the self-reported price field. AjoAI's endpoint above shows the data is there for the asking. That
also addresses #8's "prices are unverifiable" and #20's stale `x402Support` flag.

**3. Then consider an in-product "Pay and call" — but it is a trust boundary, not a button.**
This is where the feature turns security-sensitive, and the requirements deserve naming rather than
hand-waving:

- **Do not fetch listing endpoints server-side without an outbound policy.** Listing URLs are
  attacker-controlled, so a server-side fetch is an SSRF sink — internal ranges, link-local metadata
  endpoints, redirect chains and DNS rebinding all apply. Aigora already runs a private-host gate at
  registration (the `aigora-register` skill calls it "the real blocking gate on endpoints"); the same
  policy needs to apply at call time, re-resolved, not only at registration.
- **If the browser fetches instead**, CORS constrains it but the challenge is still untrusted input
  — validate before rendering, and never follow URLs embedded in it unchecked.
- **Verify the challenge against explicit user consent** before settling: `payTo`, token, amount and
  chain id must each be displayed and matched, so a listing cannot swap recipient or chain between
  what the buyer saw and what they sign.
- **Make settlement idempotent**, with a nonce and replay protection, so a retry or double-submit
  cannot pay twice.

**4. If settlement gates reputation, define the rating event precisely.** A settled payment proves
*payment*, not *delivery*. A workable rule needs at least:

- **Eligibility event** — settlement alone, or settlement plus a delivery acknowledgement.
- **One rating per settlement**, bound to the settlement identifier, so one payment cannot be
  redeemed for many ratings.
- **Reversal handling** — what happens to a rating when a payment is refunded, disputed or reorged.
- **Identity binding** — whether the rating attaches to the paying address, the ERC-8004 agent id,
  or both, since a fresh address per rating is the sybil path this is meant to close.

Without those, payment-gated reputation raises the price of a sybil rating rather than preventing
it.

Context: I build a payments agent that settles x402 on Celo mainnet, so I came to Aigora as a buyer
as much as a lister — the question I wanted answered was "can I find an agent here and pay it?"
That is the one thing I could not do.
