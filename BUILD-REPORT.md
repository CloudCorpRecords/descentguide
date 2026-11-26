# BUILD REPORT — descentguide

Clean-room rebuild of `com.eligiblebydescent/site` v1.0.1
(remote-only MCP server at `https://eligiblebydescent.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/descentguide` 0.1.0, registry name
`io.github.CloudCorpRecords/descentguide`.

## Surface built

Labelled subset: **5 of 6 captured tools built**, 1 omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `checker_start` | (none — no `required` key) |
| `checker_answer` | `question`, `choice` |
| `checker_tree` | (none — no `required` key) |
| `enquiry_describe` | (none — no `required` key) |
| `enquiry_fields` | (none — no `required` key) |

**OMITTED: `submit_enquiry`** — it submits a person's answers
and contact details to human citizenship and immigration
advisers (up to three firms chosen by the vendor, after a
consent + confirmation-token flow): a PII lead submission to
human providers. It is absent from the served surface, and
the test suite asserts that absence at registry level (not in
`TOOL_DEFS` / `TOOL_NAMES`, `isKnownTool` false), MCP level
(tools/list has exactly the 5 served tools; a tools/call for
`submit_enquiry` is rejected as unknown with MethodNotFound
before any network hop), and CLI level (`submit_enquiry` is
not a valid subcommand → usage exit 2). The two enquiry
documentation reads about that flow (`enquiry_describe`,
`enquiry_fields`) are kept.

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured: the
capture's tools carry `title` keys, per-tool `$schema` keys
inside `inputSchema`, and an `additionalProperties` key on
`checker_answer` only — all served verbatim from the fixture.
The tool literals were generated programmatically from
`harness/fixtures/eligiblebydescent-tools.json` by the script
at `/tmp/b52-descentguide-scratch/gen.py` (parse +
re-serialise; the generator's round-trip assertions confirmed
the emitted literal parses back deep-equal to the fixture's
tool objects) — never retyped — and the registration-integrity
test suite plus the parity harness confirmed the served
literals deep-equal the fixture's tool objects. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tool names,
titles, descriptions, required fields and property shapes for
all 5 served tools, including the per-tool `$schema` and
`additionalProperties` keys.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://eligiblebydescent.com/mcp`.
- Env override `DESCENTGUIDE_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote and
  the capture was taken keyless: no auth header is ever sent.
  Tests assert that no `Authorization` header is sent under
  any environment — including with key-like env vars
  (`DESCENTGUIDE_API_KEY`, `API_KEY`, `AUTHORIZATION`) set —
  and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when
  issued, `Mcp-Session-Id`.
- Error rendering `descentguide error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call
  is forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued;
  plain-JSON and SSE replies are both parsed. The forwarding
  core takes an injectable fetch; tests stub the upstream —
  no live network calls in tests.

## Deviations

**None.**

## Lockfile

- Route: Route 3 (offline) succeeded on the first attempt:
  `npm install --package-lock-only --ignore-scripts --offline`,
  generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile.
- Parses as JSON; `name` == `@reneturcios/descentguide`.
- Package entries: **128**.
- sha256: `a9e788492c617c372264fcfc86e73ce253925441e44c214e433b5406a0c4015c`
  (re-verified unchanged after the subsequent full `npm install`).

## server.json

- Description exactly: "Portuguese citizenship-by-descent route guide: the decision checker plus enquiry facts, read-only." —
  **98 characters** (verified programmatically from the file).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/descentguide:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/descentguide"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/descentguide`
  and its smoke test asserts the server name `descentguide`.

## Verification (all run 2026-10-28)

1. `tsc --noEmit` (direct binary) → exit 0.
2. `tsc` build → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. Tests (`tsx --test test/forward.test.ts`, fake-fetch only) →
   **60 tests, 60 pass, 0 fail** (registration-integrity suite
   deep-equals the served tools/list against the fixture's 5
   tools, including title/description verbatim checks,
   required-key exactness, the per-tool `$schema` /
   `additionalProperties` checks and the checker_answer
   property shapes; the submit_enquiry absence suite at
   registry, MCP and CLI level; session handling/reuse, env
   override, the no-Authorization / no-X-API-Key assertions
   under any environment, verbatim argument forwarding for all
   5 tools and result pass-through including structuredContent
   and upstream isError, SSE parsing, error rendering for both
   the JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including a working
   subcommand for each of the 5 tools against a local stub
   server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/eligiblebydescent-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/descentguide/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/descentguide/parity-out` → **verdict PASS, parity 100.0**
   (5/5 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
