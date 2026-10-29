# @pipeworx/mgnify

MGnify (EMBL-EBI) — public metagenomics studies, the controlled biome
vocabulary they are classified under, and the ~57,000 MAGs and isolate genomes
in MGnify's species-level genome catalogues, with completeness, contamination
and GTDB taxonomy for each.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `mgnify_search_studies(search?, biome_lineage?, limit?)` — microbiome studies
  by free text and/or biome, each with its ENA project accessions.
- `mgnify_study(accession)` — one MGYS study in full.
- `mgnify_genomes(search?, catalogue_id?, taxon_lineage?, limit?)` — the genome
  catalogues, with CheckM completeness/contamination, N50, GC, origin and
  ENA/NCBI cross-references.
- `mgnify_biomes(lineage_prefix?, limit?)` — the ~492-lineage biome vocabulary.
  The exact lineage strings are not guessable; look them up here first.

## Auth

Keyless. No registration step.

## Data sources

- <https://www.ebi.ac.uk/metagenomics/api/v2/studies/> — study collection.
- <https://www.ebi.ac.uk/metagenomics/api/v2/genomes/> — genome catalogues.
- <https://www.ebi.ac.uk/metagenomics/api/v2/biomes/> — biome vocabulary.

### Things that cost time to rediscover (measured 2026-09-17)

- **This pack targets v2 deliberately.** v1 is still up and still serves a
  study list, but `GET /v1/studies/MGYS00010519` returns `Not found` for a
  study v2 serves in full. A v1-based pack would answer "that study does not
  exist" about studies that do.
- **Most query filters are silently ignored.** `catalogue_id=`,
  `taxon_lineage=`, `biome_name=` and `lineage=` all return the FULL unfiltered
  collection — same count, same first row, no error. Only `search=` narrows
  anything (studies 5,684 → 532 for "gut"; genomes 56,782 → 787 for
  "Prevotella"). This pack passes `search` through and applies the rest
  client-side, reporting `rows_scanned` and `scan_exhausted` so a caller can
  tell "no matches" from "we stopped looking".
- **`search=` does not work on `/biomes` either.** The 492-row vocabulary comes
  back whole whatever you ask, so `mgnify_biomes` pages it and filters here.
- **Collection URLs need a trailing slash** (`/v2/studies/?…`; without it, 301).
  Detail URLs must NOT have one (`/v2/studies/MGYS00010519`).
- **`page_size` caps at 100 and clamps silently** — asking for 500 returns 100
  with no indication the request was reduced.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mgnify": {
      "url": "https://gateway.pipeworx.io/mgnify/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mgnify/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/mgnify_search_studies \
  -H 'Content-Type: application/json' \
  -d '{"search":"gut","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mgnify_search_studies`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mgnify` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mgnify/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mgnify": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mgnify"]
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
ask_pipeworx({ question: "your question about Mgnify data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
