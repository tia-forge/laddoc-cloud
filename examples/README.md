# Client setups

## Claude Code

```bash
claude mcp add --transport http lad-cloud https://tia.hubwiz.com/lad/mcp \
  --header "Authorization: Bearer lad_live_YOUR_KEY"
claude mcp list          # expect: lad-cloud … connected
```

## Cursor

`~/.cursor/mcp.json` (or project `.cursor/mcp.json`):

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

## VS Code (Copilot / MCP-aware builds)

`.vscode/mcp.json`:

```json
{
  "servers": {
    "lad-cloud": {
      "type": "http",
      "url": "https://tia.hubwiz.com/lad/mcp",
      "headers": { "Authorization": "Bearer ${input:lad_key}" }
    }
  },
  "inputs": [
    { "id": "lad_key", "type": "promptString", "description": "lad_live_… API key", "password": true }
  ]
}
```

## Troubleshooting

| Symptom | Cause |
|---|---|
| `401` on the first request | Missing/expired key, or the `Bearer ` prefix was dropped |
| `404` | Wrong path — the endpoint is `/lad/mcp`, not `/mcp` |
| `lad_generate` returns no XML | Validation failed on purpose: read `issues[]`, fix, retry |
| “which spec version am I being judged against?” | Call `lad_version` (free) and compare `bundleHash` with the LAD-IR spec service |
