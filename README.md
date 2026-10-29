# @pipeworx/fra-rail-safety

US railroad accidents, derailments and highway-rail grade crossing collisions
reported to the Federal Railroad Administration, 1975 to the present: railroad,
FRA cause code, location, casualties and damage per report, plus grouped counts.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `rail_accidents(year?, state?, railroad?, cause_code?, accident_type?, min_damage_usd?, limit?)`
  — individual rail equipment accident reports (derailments, collisions,
  obstructions, fires), newest first, with the reporting railroad, accident
  type, FRA primary and contributing cause code and text, state/county, train
  speed, hazmat cars released, people evacuated, killed, injured, damage cost
  and a link to FRA's own report page.
- `crossing_incidents(state?, county?, city?, year?, railroad?, crossing_id?, highway_user?, fatal_only?, limit?)`
  — collisions between trains and road users at railroad crossings, newest
  first, with the USDOT crossing id, road name, road user type and action,
  vehicle and train speed, warning devices, weather, killed and injured.
- `rail_accident_summary(group_by, dataset?, year?, from_year?, state?, railroad?, limit?)`
  — counts and casualty/damage totals grouped by railroad, cause, accident
  type, state, year or road user, for either dataset.

## Auth

Keyless.

## Data sources

- Federal Railroad Administration, Office of Safety Analysis — Rail
  Equipment Accident/Incident Data (Form 54).
- Federal Railroad Administration, Office of Safety Analysis — Highway-Rail
  Grade Crossing Incident Data (Form 57).

Both are published by the US Department of Transportation and queried live on
each call. US federal government work, public domain.

Things worth knowing:

- **One row per railroad report, not per accident.** An accident involving two
  railroads (joint operations, trackage rights) is filed once by each. FRA's
  unique-accident count keys on the incident key; `rail_accident_summary`
  reports both `accidents` (distinct) and `railroad_reports`.
- **Railroad filter:** 1-4 character input is matched against the reporting
  mark exactly (`NS`, `UP`, `BNSF`, `CSX`, `ATK`); longer input is a name
  substring (`Norfolk Southern`). A substring match on a reporting mark is
  wrong — `NS` occurs inside "CSX TRANSPORTATION".
- **State fields differ between the datasets:** accidents carry a two-letter
  code, crossings only the upper-case full name. The pack accepts either form
  and translates.
- **Lag:** railroads file monthly and FRA publishes a few months behind, so the
  current year is partial.
- Cause codes are FRA's: the first letter is the family (T track, S signal,
  H human factors, E equipment, M miscellaneous); `cause_code` matches a prefix.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "fra-rail-safety": {
      "url": "https://gateway.pipeworx.io/fra-rail-safety/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/fra-rail-safety/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/rail_accidents \
  -H 'Content-Type: application/json' \
  -d '{"year":2023,"state":"OH","railroad":"NS","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/rail_accidents`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-fra-rail-safety` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/fra-rail-safety/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "fra-rail-safety": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-fra-rail-safety"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Fra Rail Safety data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
