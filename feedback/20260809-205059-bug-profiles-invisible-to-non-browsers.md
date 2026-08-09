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
Agent profile pages return no server-rendered content. Every route on aigora.org serves the same
3,689-byte JavaScript shell, so anything that does not execute JS — link-preview bots, search
crawlers, and other AI agents — sees an empty page with a generic site-wide title.

The page renders correctly in a browser. This is not about human visitors. It is about every
non-browser consumer of the URL, which for a marketplace is most of them.

Verified against my own registered agent:

- The profile HTML and the homepage HTML are **byte-identical** — same MD5
  (`5f7d8925f83e68cdd94b14e8f92b51c5`), 3,690 bytes each.
- The string "AbaPay" appears **zero** times in the profile HTML. So do "9760", "bill" and
  "payment".
- There are **no** `og:` tags, **no** `twitter:` tags, no `<meta name="description">`, and no
  `<link rel="canonical">`.
- `<title>` is `Aigora — Agentic Conversation & Agent Marketplace` on the profile page — the same
  title the homepage serves. Every agent profile is titled identically.

I checked whether crawlers get prerendered HTML instead (a common setup). They do not — same
empty shell for all of them:

| User-Agent | Bytes | "AbaPay" hits | `og:` tags |
|---|---|---|---|
| `Googlebot/2.1` | 3,689 | 0 | 0 |
| `Twitterbot/1.0` | 3,689 | 0 | 0 |
| `facebookexternalhit/1.1` | 3,689 | 0 | 0 |
| `Slackbot-LinkExpanding 1.0` | 3,689 | 0 | 0 |

Three consequences, in rising order of how much they matter to Aigora specifically:

**1. Sharing a profile produces no preview.** Post your agent link on X, Telegram, Discord or
Slack and you get a bare URL with the generic marketplace title. Every agent on Aigora unfurls
identically — same title, no description, no image. The hackathon asks builders to submit and
share this URL as proof they are listed, so this is the single most-shared link the product has,
and it is the one that carries no information.

**2. Individual agents cannot be found by search.** With no crawlable per-agent content and no
per-agent title or description, agents are not indexable. Organic discovery of a specific agent
via search is not possible today.

**3. Other agents cannot read the marketplace.** This is the one I would prioritise. Aigora is an
*agent* marketplace built on ERC-8004 — the premise is that agents discover and transact with
each other. But an agent doing discovery typically fetches a URL and parses what comes back; it
does not run a headless browser. Right now a peer agent that fetches an Aigora profile receives
an empty shell and learns nothing — not the name, not the description, not the service endpoints.
The marketplace is machine-unreadable to precisely the machines it exists to connect.

Related, and probably the same root cause — a catch-all that returns `index.html` for every path:

- `https://aigora.org/robots.txt` returns the HTML shell with
  `Content-Type: text/html; charset=utf-8`. A crawler asking for robots directives receives a
  web page, so there are effectively no directives.
- `https://aigora.org/sitemap.xml` returns 200 but is also the HTML shell, also
  `Content-Type: text/html`. A sitemap that is not XML cannot be consumed — and a sitemap is the
  standard fix for exactly the indexing problem above.
- Nonexistent agents return **200, not 404**. Both
  `/services/42220_0x8004a169fb4a3325136eb29fa0ceb6d2e539a432_99999999` (valid shape, no such
  agent) and `/services/not-a-real-agent` (garbage) return 200 with the same shell. A typo'd or
  removed agent URL is indistinguishable from a real one to any automated consumer.

Suggested fixes, cheapest first:

1. **Server-render per-agent `<title>`, `<meta name="description">`, and `og:`/`twitter:` tags**
   on `/services/:id`. The data is already pinned in `agent.json` and readable on-chain — name,
   description and image are all right there. This alone fixes link previews and most of the
   indexing problem, without converting the whole app to SSR.
2. **Serve real `robots.txt` and `sitemap.xml`** as static files ahead of the SPA catch-all, with
   the sitemap enumerating registered agents.
3. **Return a real 404** for unknown agent ids.
4. **Offer machine-readable profile output** — content negotiation on `Accept: application/json`,
   or a documented `/api/agents/:id` — so a peer agent can read a listing without scraping. Given
   the ERC-8004 metadata already exists as JSON, this is mostly plumbing, and it is what turns
   "agent marketplace" from a description of the users into a description of the API.

### Steps to reproduce
```shell
U="https://aigora.org/services/42220_0x8004a169fb4a3325136eb29fa0ceb6d2e539a432_9760"

# 1. Profile HTML contains nothing about the agent
curl -s "$U" | grep -c "AbaPay"          # -> 0
curl -s "$U" | grep -ciE 'og:|twitter:'  # -> 0

# 2. Profile and homepage are the same file
curl -s "$U" -o profile.html; curl -s https://aigora.org -o home.html
md5sum profile.html home.html            # -> identical

# 3. Crawlers get no prerender either
curl -s -A "Twitterbot/1.0" "$U" | grep -c "AbaPay"   # -> 0

# 4. robots.txt / sitemap.xml are HTML
curl -sI https://aigora.org/robots.txt  | grep -i content-type   # -> text/html
curl -sI https://aigora.org/sitemap.xml | grep -i content-type   # -> text/html

# 5. Nonexistent agent returns 200
curl -s -o /dev/null -w "%{http_code}\n" https://aigora.org/services/not-a-real-agent   # -> 200
```

### Logs / console output
_No response_

### Transaction / agent ID
Agent `9760` — Celo mainnet, identity registry
`0x8004A169FB4a3325136EB29fA0ceB6D2e539a432`

### Anything else
Worth saying that the underlying registration is sound — the metadata really is pinned and really
is on-chain, so all the data needed to fix this is already there. Nothing here requires new data
collection or a schema change; it is a rendering and routing gap between correct on-chain state
and what the public URL actually serves.
