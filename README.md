# ConferenceGrid MCP server

<img src="logo.png" alt="" width="96" align="right">

Ask Claude, ChatGPT or any MCP client about B2B conferences and get answers
from a catalogue that re-verifies itself daily, instead of from whatever the
model remembers.

**Endpoint**

```
https://conferencegrid.com/api/mcp
```

Use the apex domain exactly as written. The `www` form redirects, and most
clients turn a redirect into a request this endpoint refuses, which looks like
a broken server rather than a wrong address.

Full documentation: **https://conferencegrid.com/mcp**

---

## What it answers

| tool | answers |
|---|---|
| `search_conferences` | upcoming events by city, country, topic, date window |
| `get_conference` | one event with its speakers, sponsors, exhibitors, sessions |
| `company_events` | every event a company appears at — the one nothing else answers |
| `events_open_for_sponsorship` | who is taking sponsors now, plus the peers a company is missing |
| `find_calls_for_speakers` | open calls with deadlines and submission links |
| `organizer_contact` | how to reach an organiser, as a link |
| `whats_new` | what changed for a company or conference since a date |
| `search` / `fetch` | the ChatGPT connector contract, over the same data |

Nine tools, all read-only. Nothing here can write, delete or spend money.

Questions it handles well:

> Which conferences does Datadog sponsor in the next six months?

> Find fintech conferences in Berlin before June, and tell me which have a sponsor prospectus.

> Which security conferences are still open for speaker submissions?

---

## Connect it

Every client does the same three things: paste the URL, sign in to
ConferenceGrid in the browser window that opens, click **Allow**. Sign-in is a
one-time link sent to your email, and an account is created on first sign-in.
There is no API key.

**Claude** (web, desktop): Settings → Connectors → Add custom connector. Paste
the URL and leave the OAuth fields empty; Claude discovers them.

**Claude Code**

```bash
claude mcp add --transport http conferencegrid https://conferencegrid.com/api/mcp
```

then `/mcp`, pick `conferencegrid`, choose **Authenticate**.

**ChatGPT**: add it as a custom connector with the same URL. The server ships
`search` and `fetch` on OpenAI's contract alongside the domain tools.

**Anything else** that speaks Streamable HTTP and OAuth 2.1 with PKCE: point it
at the URL. Discovery is standard — an unauthenticated call returns `401` with
a `WWW-Authenticate` header naming the protected-resource metadata, which names
the authorization server at `/.well-known/oauth-authorization-server`. Clients
identify with a Client ID Metadata Document URL or through dynamic
registration. Public clients only, S256 only.

Check it is up:

```bash
curl -si -X POST https://conferencegrid.com/api/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | head -5
```

A `401` carrying `WWW-Authenticate: Bearer resource_metadata=...` is the healthy
answer: the server is up and asking you to sign in.

---

## Before you trust an answer

**Every record carries the date it was last verified.** That is the point of
the product. A number with a date on it is a different claim from a number
without one, so ask for it if the model does not volunteer it.

**An empty list is an answer, not a failure.** About one upcoming edition in
five publishes before its line-up does. "No sponsors listed" means we looked
and the organiser has not published them yet.

**Coverage thins the further out you look, and that is seasonal.** The
catalogue holds roughly one year per conference, and organisers publish next
year's dates after this year's event runs. Compare a month to the same month a
year earlier, never to the month beside it.

**Organiser emails never leave the server.** `organizer_contact` returns a link
to the event page, never an address, and it declines events less than 21 days
out.

**Scraped text is sanitised before it reaches your model.** Every description
and biography came from someone else's website, and all of it is stripped of
markup and instruction-shaped framing on the way out, because an MCP server
piping raw scraped prose into your context is an injection path.

---

## Limits

- 60 calls a minute per account.
- Every tool caps the rows it returns and states the cap in its own
  description. There is no way to dump the catalogue.
- `whats_new` reports records that **appeared** and editions that were
  **edited**. It cannot yet report an edited record, so a sponsor moving from
  silver to gold is invisible to it.

## Something wrong?

The data is read from conference websites, so some of it will be wrong. Report
a bad record through the [contact form](https://conferencegrid.com/contact)
with the event name. A wrong record reported is more use to us than a polite
silence.

---

This repository is the public home of the connector: documentation, the
registry manifest (`server.json`) and the mark. The server itself is hosted by
ConferenceGrid, so there is nothing here to install.

Listed in the [official MCP Registry](https://registry.modelcontextprotocol.io)
as `com.conferencegrid/conferencegrid`.
