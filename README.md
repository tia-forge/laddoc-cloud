# LadDoc Cloud — LadDoc → SimaticML over MCP

**LadDoc Cloud** turns a ladder-logic document (LadDoc IR) into **SimaticML** that TIA Portal
imports directly — with a strict validation gate in front of the generator.

> This repository is **documentation and client configuration only**. The service runs at
> `tia.hubwiz.com`; its implementation is not open source.

- **Endpoint**: `https://tia.hubwiz.com/lad/mcp`
- **Transport**: Streamable HTTP (stateless, POST JSON-RPC)
- **Auth**: `Authorization: Bearer lad_live_…` (the `X-API-Key` header is also accepted)
- **Dashboard / API keys**: https://tia.hubwiz.com/lad
- **Registry entry**: `io.github.tia-forge/laddoc-cloud` ([official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=laddoc-cloud))

## What it does

| Tool | Purpose |
|---|---|
| `lad_version` | Free: returns the LAD-IR identity this service implements (`schemaVersion`, `bundleHash`, `sourceCommit`) |
| `lad_validate` | Validate one LadDoc document; returns `valid` plus `issues` with `code` / `path` / `message` |
| `lad_generate` | Generate a SimaticML block (FC / FB / OB) — **only if validation passes** |

Behaviour worth knowing:

- **Validation is the gate.** One issue and no XML is produced — you never hand TIA Portal a block
  that cannot be imported. Fix and retry; the remaining issues come back in full.
- **Deterministic output.** The same LadDoc always produces byte-identical SimaticML; `UId`s and
  connections are assigned by the generator.
- **Target format**: V19 SimaticML documents.
- **Optional cross-check**: pass the target project's inventory to catch symbol mismatches before
  generation.
- One document per call: to generate several blocks, call once per block (called FC/FBs are
  submitted as their own documents).

## Quick start

### Claude Code

```bash
claude mcp add --transport http lad-cloud https://tia.hubwiz.com/lad/mcp \
  --header "Authorization: Bearer lad_live_YOUR_KEY"
claude mcp list
```

### Cursor / VS Code / any JSON-configured client

```json
{
  "mcpServers": {
    "lad-cloud": {
      "url": "https://tia.hubwiz.com/lad/mcp",
      "headers": { "Authorization": "Bearer lad_live_YOUR_KEY" }
    }
  }
}
```

### First calls

```jsonc
// free — confirm which LAD-IR bundle this service implements
{ "name": "lad_version", "arguments": {} }

// validate, then generate
{ "name": "lad_validate", "arguments": { "doc": { "schemaVersion": "1.0", "block": { /* … */ } } } }
{ "name": "lad_generate", "arguments": { "doc": { /* same doc */ } } }
```

## Pricing

WeChat login on the dashboard issues a key immediately: **100 calls free**. Handshake requests
(`initialize`, `tools/list`) and `lad_version` are free.

## Notes

- This service produces **text/XML**; it does not touch a running TIA Portal. To import the result
  and compile it, use TIA Portal itself or the on-machine gateway
  ([TiaLink](https://github.com/tia-forge/tialink)).
- Free companion: [PLC Cloud Sim](https://github.com/tia-forge/plc-cloud-sim) runs the logic before
  you import it.
- Documentation license: CC BY 4.0. "TIA Portal", "SIMATIC" and "S7" are trademarks of Siemens AG;
  this service is an independent third-party tool, not affiliated with or endorsed by Siemens.
