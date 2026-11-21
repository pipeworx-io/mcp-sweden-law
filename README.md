# @pipeworx/sweden-law

Swedish national legislation: search the Svensk forfattningssamling (SFS, the Swedish Code of Statutes) by word, and fetch one statute or a single section, with the SFS citation, the repeal status the source states, and how far the consolidated text is amended.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2020+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `se_law_search(query, include_repealed?, year_from?, year_to?)` — statutes matching a Swedish word, best matches first (a word in the title ranks first). Returns SFS number, title, issue date, ministry, repealed flag, amended-through marker and link. Repealed statutes are hidden unless `include_repealed=true`.
- `se_law_get(id, section?, max_chars?)` — one statute by SFS number ("2018:218"), text in Swedish. `section` ("5", "3 a" or "2:1" for chapter 2 section 1) returns a single section.

## Auth

Keyless.

## Data sources

- Sveriges riksdag, Svensk forfattningssamling (SFS), open data. Live call per request, nothing stored.

The source states whether a statute is repealed (with the repeal date and the repealing act) and up to which amending act the text is consolidated ("t.o.m. SFS ..."). It states no positive in-force flag, so none is derived: `repealed: false` means the Riksdag marks no repeal, nothing more. Scheduled changes appear inside the text as the Riksdag's own marker lines ("/Träder i kraft I:2027-01-01/"); a section with such a change returns every version. The text is the Riksdag's consolidated text, not the government's authoritative publication.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sweden-law": {
      "url": "https://gateway.pipeworx.io/sweden-law/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sweden-law/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2020+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/se_law_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"dataskydd"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/se_law_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-sweden-law` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/sweden-law/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "sweden-law": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sweden-law"]
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
ask_pipeworx({ question: "your question about Sweden Law data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
