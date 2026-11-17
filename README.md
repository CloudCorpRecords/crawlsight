# crawlsight

How visible is a website to AI search? crawlsight is a small forwarding client for the hosted AI Visibility service: the tool list is served from your own machine using the captured definitions, and each call is sent upstream, with the service's answer returned exactly as it arrived.

All four captured tools are exposed. You can run a full visibility check on a site, generate a fresh llms.txt for it from its real pages, validate the llms.txt it already ships, and lint its robots.txt for the kinds of mistakes that quietly block AI crawlers. Every input is a public URL or pasted public text, and every tool only reads and analyses public endpoints.

## The tools

- **`check_site`** — Check a site's AI-search visibility: which AI crawlers (OAI-SearchBot, PerplexityBot, GPTBot, ClaudeBot and more) its robots.txt allows, whether the homepage ships readable text without JavaScript, and whether an llms.txt exists. Takes a required `url`.
- **`generate_llms_txt`** — Generate an llms.txt file from a site's real pages and sitemap. Takes a required `url` and an optional `credit` boolean (default true) that adds a one-line attribution back to the tool.
- **`validate_llms_txt`** — Validate an existing llms.txt against the spec. Takes a required `url`; the site's `/llms.txt` is fetched and validated.
- **`lint_robots_txt`** — Lint a robots.txt for AI-crawler-specific mistakes. Pass `url` to fetch it live, or `robots_txt` to lint pasted text directly. Neither field is required.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/crawlsight mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "crawlsight": {
      "command": "npx",
      "args": ["-y", "@reneturcios/crawlsight", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/crawlsight
docker run --rm ghcr.io/cloudcorprecords/crawlsight check_site '{"url":"https://example.com"}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
crawlsight check_site '{"url":"https://example.com"}'
crawlsight generate_llms_txt '{"url":"https://example.com","credit":false}'
crawlsight validate_llms_txt '{"url":"https://example.com"}'
crawlsight lint_robots_txt '{"url":"https://example.com"}'
crawlsight --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `AIVISIBILITY_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://ai-visibility.lastminutedealshq.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The AI Visibility registry record lists no headers for its remote, so crawlsight never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `crawlsight error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, crawlsight repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
