# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Eligible By Descent MCP server, serving a labelled 5-of-6 subset of the captured surface. The 5 served tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `checker_start`, `checker_answer`, `checker_tree`, `enquiry_describe`, `enquiry_fields`. Each tool's `title` is served exactly as captured, as are the per-tool `$schema` and `additionalProperties` keys inside each inputSchema.
- Omitted: `submit_enquiry` — the upstream tool that submits a person's contact details to human citizenship and immigration advisers. It is absent from the served surface at registry, MCP and CLI level.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `descentguide error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `DESCENTGUIDE_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
