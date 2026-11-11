# HopBase MCP server

[HopBase](https://hop-base.com) is an AI gateway that serves Claude, GPT, Gemini, DeepSeek and other text, image and video models through one base URL. This MCP server lets a coding agent look up what its HopBase API key can do: which models it can call, their limits, its balance and its usage.

- **Endpoint:** `https://api.hop-base.com/mcp`
- **Transport:** Streamable HTTP, stateless, POST only (one JSON-RPC message per request; no SSE stream or batch requests)
- **Authentication:** `Authorization: Bearer <key>` or `x-api-key: <key>`, with the same API key you use for model calls. Required to call a tool; listing the tools needs no key
- **Scope:** read-only; every result covers only the key in use

This is a hosted service and its source is not published. This repository holds the MCP registry manifest ([`server.json`](server.json)) and client configs.

## Tools

| Tool | Returns |
| --- | --- |
| `list_models` | Models this key can call, with effective prices |
| `get_model_limits` | For one model ID: whether this key can call it, plus the endpoints, request parameters, and limits documented for it, with links to the source pages |
| `search_docs` | The docs sections that best match a query (model ID, parameter name, error code), each with URL and snippet; `lang` selects en, zh-cn, zh-tw, es or ja |
| `get_balance` | Account balance and this key's spend-cap usage, in USD |
| `get_key_info` | Key name, groups, spend cap, concurrency cap, expiry, and status |
| `get_usage` | Requests, tokens, and billed cost for this key; last 7 days by default, up to 92 days |

`get_model_limits` and `search_docs` read the published docs and refresh at most once an hour. The linked docs page is authoritative: a limit missing from a result does not mean there is none.

## Setup

You need a HopBase API key: create one in the [console](https://api.hop-base.com) and export it as `HOPBASE_API_KEY`.

### Claude Code

```bash
claude mcp add --transport http hopbase https://api.hop-base.com/mcp \
  --header "Authorization: Bearer $HOPBASE_API_KEY"
```

The shell expands the variable when you run the command, so the key is saved in your local Claude Code configuration. For a shared project configuration, put [`examples/claude-code.mcp.json`](examples/claude-code.mcp.json) in the project as `.mcp.json` instead; Claude Code expands `${HOPBASE_API_KEY}` at startup, so the file holds no key.

### Cursor

Save [`examples/cursor.mcp.json`](examples/cursor.mcp.json) as `.cursor/mcp.json` in the project, or as `~/.cursor/mcp.json` for every project:

```json
{
  "mcpServers": {
    "hopbase": {
      "url": "https://api.hop-base.com/mcp",
      "headers": { "Authorization": "Bearer ${env:HOPBASE_API_KEY}" }
    }
  }
}
```

### Other clients

Any client that supports remote Streamable HTTP servers with a custom header can connect: point it at the endpoint and send the `Authorization` header above.

## Good to know

- **The key decides what MCP sees.** `list_models` and `get_model_limits` see only the groups of the key you configure. To inspect another model family, add its group to the key, or register a second server entry with a key that includes it.
- **Model calls do not go through MCP.** Keep inference on the regular API endpoints (`https://api.hop-base.com/v1`: OpenAI Chat Completions and Responses, Anthropic Messages, native Gemini, and the image and video task endpoints).
- **Calling a tool needs a key.** The handshake (`initialize`) and `tools/list` answer without credentials, so clients and directories can discover the tools; `tools/call` without a key returns 401.

## Links

- Coding-agent guide and MCP docs: https://hop-base.com/en/docs/getting-started/ai-agent
- Docs: https://hop-base.com/en/docs
- Machine-readable integration contract: https://hop-base.com/llms.txt
- Support: support@hop-base.com

## License

The files in this repository are released under the [MIT License](LICENSE).
