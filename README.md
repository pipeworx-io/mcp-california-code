# California Code — California statutes by citation

The text of California's 29 codes. Give it `PEN` and `187`, get back the murder
statute.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Auth

None. Both tools are keyless.

## Why this isn't a mirror

The probe that led to this pack (`docs/state-law-probe.md`) found California
publishing a ~1.0 GB bulk drop *daily* and sized the work as a
CourtListener-style mirror. It doesn't need to be one — leginfo serves any
section directly:

```
?lawCode=PEN&sectionNum=187  →  "187. (a) Murder is the unlawful killing…"
```

So this is the `us-code` shape, not the CourtListener shape: no storage, no
recurring cost, no quarterly refresh obligation. That matters well beyond
California — it means the other 49 states are 49 *small* packs rather than 49
mirrors, which changes whether the long tail is worth building at all.

## Tools

| Tool | Returns |
|---|---|
| `ca_code_section` | Full statutory text, where it sits in the code, and when it was last amended |
| `ca_codes` | The 29 codes and their subjects, filterable ("divorce" → `FAM`) |

```
ca_code_section({code: "PEN", section: "187"})
  → located_in: PART 1 → TITLE 8 → CHAPTER 1. Homicide
    last_affected: Amended by Stats. 2023, Ch. 260, Sec. 14. (SB 345)
```

## Caveats worth passing on

- **The upstream fails silently, and this pack does not.** A nonexistent
  section returns HTTP 200 with a kilobyte of page furniture and no statute —
  no error, no 404. We detect the missing content container and return an
  explicit `section_not_found`. Treating that page as an answer would report
  "no such law" as though it were the law.
- **Subdivisions are not separate documents.** `187(a)` renders the same empty
  shell upstream, so it's stripped to `187` and the response says so.
- **Section numbers are not contiguous.** Repealed and reserved numbers are
  normal; a miss is usually a real gap rather than a bad request.
- **`last_affected` is per section**, which is better currency than the federal
  code can offer — `us-code` can only tell you which annual edition the text
  came from.

## Related

- `us-code` — federal statutes, the same citation-resolver shape
- `court-listener` — California cases interpreting these sections, with full text
- `recap` — federal filings

## Data source

<https://leginfo.legislature.ca.gov/> — California Legislative Information.
California statutes are public record.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "california-code": {
      "url": "https://gateway.pipeworx.io/california-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/california-code/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "california-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-california-code"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-california-code
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about California Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
