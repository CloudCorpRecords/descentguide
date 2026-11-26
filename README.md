# descentguide

Which Portuguese citizenship-by-descent route applies to your family? descentguide is a local forwarding client for the hosted Eligible By Descent MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

This package serves a 5-tool subset of the upstream surface: the decision checker trio and the two enquiry documentation reads. The upstream's enquiry submission tool is not part of this package — nothing here sends a person's contact details to anyone.

## The tools

- **`checker_start`** — The first question of the "Which Portuguese descent route?" decision guide. No arguments.
- **`checker_answer`** — Given a `question` id and the chosen option's `choice` index, returns the next question or the guide's final verdict. Both `question` and `choice` are required.
- **`checker_tree`** — Every question, option and verdict of the whole decision tree, for reasoning end to end. No arguments.
- **`enquiry_describe`** — States plainly what the upstream enquiry submission does: it starts an enquiry with human providers who quote directly, nothing is bought or paid, no quote is guaranteed, and it is free. Also returns who receives the details, the consent wording, and how the person confirms. No arguments.
- **`enquiry_fields`** — Every field the upstream enquiry asks: key, label, type, whether required, help text and allowed options. No arguments.

The checker collects no personal details and commits nothing: it walks a published decision guide over a question id and a choice index. The enquiry tools are documentation reads about the submission flow, not the submission itself.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/descentguide mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "descentguide": {
      "command": "npx",
      "args": ["-y", "@reneturcios/descentguide", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/descentguide
docker run --rm ghcr.io/cloudcorprecords/descentguide checker_tree '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
descentguide checker_start '{}'
descentguide checker_answer '{"question":"q-grandparent","choice":1}'
descentguide checker_tree '{}'
descentguide enquiry_describe '{}'
descentguide enquiry_fields '{}'
descentguide --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DESCENTGUIDE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://eligiblebydescent.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `descentguide error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 5 served tools, deep-equal, titles included exactly as captured, each `required` list and the per-tool `$schema` / `additionalProperties` keys exactly as captured), asserts that the omitted `submit_enquiry` tool is absent at registry, MCP and CLI level, exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
