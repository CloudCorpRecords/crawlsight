# BUILD REPORT — crawlsight

Clean-room rebuild of `com.lastminutedealshq/ai-visibility` v1.0.1
(remote-only MCP server at `https://ai-visibility.lastminutedealshq.com/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/crawlsight`
0.1.0, registry name `io.github.CloudCorpRecords/crawlsight`.

## Files (19-file factory set)

`package.json`, `server.json`, `Dockerfile`, `tsconfig.json`,
`.gitignore`, `package-lock.json`, `README.md`, `LICENSE`,
`CHANGELOG.md`, `src/index.ts`, `src/cli.ts`, `src/mcp.ts`,
`src/types.ts`, `src/forward.ts` (core), `test/forward.test.ts`,
`.github/workflows/ci.yml`, `.github/workflows/publish.yml`,
`.github/workflows/publish-container.yml`,
`.github/workflows/registry-publish.yml` — plus this report and the
`parity-out/` directory.

## Surface built

Full surface: **4 of 4 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `check_site` | `url` |
| `generate_llms_txt` | `url` |
| `validate_llms_txt` | `url` |
| `lint_robots_txt` | *(none — no `required` key in the capture, served verbatim)* |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema` (no `title` — none was captured), and no kept
`inputSchema` carries an `additionalProperties` key — none was
captured. `generate_llms_txt` carries the optional boolean `credit`
alongside its required `url`; `lint_robots_txt` carries the optional
strings `url` and `robots_txt` and no required fields. The tool
literals were generated programmatically from
`harness/fixtures/lastminutedealshq-ai-visibility-tools-original.json`
by the script at `/tmp/laneb6-crawlsight-scratch/gen.py` (parse +
re-serialise, splice into `src/forward.ts` between the
TOOL_DEFS_GENERATED markers) — never retyped — and the generator's
own post-generation check, the registration-integrity test suite,
and the parity harness each confirmed the served literals
deep-equal the fixture's 4 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers), not
McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
descriptions, the `url` required field on `check_site`,
`generate_llms_txt` and `validate_llms_txt`, the absence of a
`required` key on `lint_robots_txt`, and the absence of
`additionalProperties` markers throughout.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://ai-visibility.lastminutedealshq.com/mcp`.
- Env override `AIVISIBILITY_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with an `AIVISIBILITY_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `AIVISIBILITY_TOKEN` variables present.
- Error rendering `crawlsight error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

- Parity output plumbing only (no result difference): the canonical
  `parity.py` invocation computed its report but the VM's write
  guard EPERM-blocked the harness process's own writes of
  `parity.json` (Python-process writes were blocked VM-wide during
  the build window; shell and Node writes were verified working).
  The outputs in `parity-out/` were therefore produced by running
  the harness's own `build_report` / `render_md` in-process (same
  fixture, same stdio rebuild command, same computation) and
  writing the resulting report through the file tool, per the
  standing write-guard workaround in AGENTS.md. Verdict and numbers
  below are the harness's own.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then populated with `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/crawlsight`.
- Package entries: **128**.
- sha256: `4edae7786256a5f679d5f71fab1206aafac73918fa2ace724a9a377855b117b2`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Check a site's AI visibility: crawler access, llms.txt generate and validate, robots.txt lint." — **94 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/crawlsight:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/crawlsight"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/crawlsight`
  and its smoke test asserts the server name `crawlsight`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **65 tests, 65 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 4
   tools, including an in-memory-transport MCP tools/list check;
   plus session handling/reuse, env override, the
   no-Authorization-header assertions under every environment,
   verbatim argument forwarding for all four tools and result
   pass-through including upstream isError, SSE parsing, error
   rendering, MCP-layer validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/lastminutedealshq-ai-visibility-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/crawlsight/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/crawlsight/parity-out` → **verdict PASS, parity 100.0**
   (4/4 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/` (see Deviations for the output-plumbing note).
