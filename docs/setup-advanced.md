# Advanced setup

Everything past the [README](../README.md) quick start: non-default clients, HTTPS, Claude Desktop, running several instances, OAuth internals, and adding a client of your own.

## Other clients

TDMCP is a standard streamable-HTTP MCP server at `http://127.0.0.1:13316/mcp`. Use the literal `127.0.0.1`; the socket is IPv4-only and `localhost` resolves to `::1` first on many systems.

The fastest route is always the component: pick your client in **Install For** and copy **Add MCP**. The commands below are what that generates, shown for reference. Client CLIs change fast, so if one is out of date, check your client's docs or ask the agent to wire itself up.

### CLI registration

Run in your project directory.

```bash
# Claude Code — project scope with --scope project
claude mcp add --transport http --scope project touchdesigner http://127.0.0.1:13316/mcp

# Codex — always global (~/.codex/config.toml)
codex mcp add touchdesigner --url http://127.0.0.1:13316/mcp

# Antigravity — always global; the scheme is detected, so no --type flag
agy mcp add touchdesigner http://127.0.0.1:13316/mcp

# OpenCode v1 — always global (~/.config/opencode/opencode.jsonc)
opencode mcp add touchdesigner --url http://127.0.0.1:13316/mcp

# OpenCode v2 beta — writes ./opencode.json; add --global for the user config
opencode2 mcp add touchdesigner --url http://127.0.0.1:13316/mcp
```

**Removal is not symmetric.** Claude Code, Codex and Antigravity all have `mcp remove`. OpenCode does not, despite having `mcp add`; delete the entry by hand. Cursor has no MCP CLI at all. For both, **Remove MCP** shows the file and the entry to edit instead of a command.

### File config

For clients with no `mcp add`, or when you want the mapping committed.

```jsonc
// Claude Code — .mcp.json in the project root (what --scope project writes)
{ "mcpServers": { "touchdesigner": { "type": "http", "url": "http://127.0.0.1:13316/mcp" } } }
```
```jsonc
// Cursor — .cursor/mcp.json in the workspace root
{ "mcpServers": { "touchdesigner": { "type": "http", "url": "http://127.0.0.1:13316/mcp" } } }
```
```jsonc
// OpenCode v1 — ./opencode.json in the project root, or the global
// ~/.config/opencode/opencode.jsonc. Note `mcp`, not `mcpServers`
{ "mcp": { "touchdesigner": { "type": "remote", "url": "http://127.0.0.1:13316/mcp" } } }
```
```jsonc
// OpenCode v2 beta — ./opencode.json in the project root, or the global
// ~/.config/opencode/opencode.json(c). Note `mcp.servers`, not `mcpServers`.
// codemode: false hands the tools to the model directly instead of through
// OpenCode's JavaScript `execute` layer, which small local models misuse
{ "mcp": { "servers": { "touchdesigner": { "type": "remote", "url": "http://127.0.0.1:13316/mcp", "codemode": false } } } }
```
```json
// Antigravity — ~/.gemini/config/mcp_config.json. Note `serverUrl`, not `url`
{ "mcpServers": { "touchdesigner": { "serverUrl": "http://127.0.0.1:13316/mcp" } } }
```
```toml
# Codex — ~/.codex/config.toml
[mcp_servers.touchdesigner]
url = "http://127.0.0.1:13316/mcp"
```

No two of these agree. The container key is `mcpServers` for most, `mcp` for OpenCode v1, `mcp.servers` for v2, and a `[mcp_servers.x]` table for Codex. The URL key is `url` everywhere except Antigravity, which needs `serverUrl` and rejects `url`. Copy the block for your client exactly.

### Which clients have a project scope

Its `mcp add` writes per-project config, and `--scope project` produces a committable `.mcp.json`. Codex writes only the global user config and does not auto-discover a project `.codex/config.toml`; `CODEX_HOME=./.codex` is the only project route. Antigravity is likewise global, with a plugin as its project route (`<project>/.agents/plugins/<name>/mcp_config.json`, registered with `agy plugin install`, appearing in the TUI's MCP Servers panel rather than in `agy mcp list`). OpenCode v1's CLI is global but it reads a project `opencode.json`. The v2 beta's `mcp add` writes a project `opencode.json` by default and the user config with `--global`. See [opencode.md](opencode.md).

### Antigravity headless

Running `agy --print` needs an allow-list in `~/.gemini/antigravity-cli/settings.json`, because headless mode cannot prompt. It denies one tool at a time, and the narrow `mcp(<server>)` form is refused where the wildcard is accepted:

```json
{ "permissions": { "allow": ["mcp(*)", "command(*)", "read_file(*)"] } }
```

Headless also does not mount workspace skills. Install at user scope — see [Skills](../README.md#skills).

## HTTPS

Optional. Claude Code connects over plain HTTP, and Claude Desktop's bridge accepts plain-HTTP localhost with `--allow-http`. Set HTTPS up when you want TLS on the wire, which `All interfaces` requires, or when your environment forbids `--allow-http`.

TDMCP uses a single user-level cert shared across every TD project, stored beside the mkcert root CA at `mkcert -CAROOT`:

- **macOS** `~/Library/Application Support/mkcert/`
- **Windows** `%LOCALAPPDATA%\mkcert\`
- **Linux** `~/.local/share/mkcert/`

Toggling **Use HTTPS** looks there and offers to generate the cert if it is missing. Once per machine; every later project picks up the same cert.

> **Generating the cert and trusting the CA are separate steps.** The **Generate** button cannot install the root CA into your OS trust store, which needs rights TouchDesigner does not have. Run `mkcert -install` yourself, once per machine, or your browser will warn on the OAuth consent page. The `mcp-remote` bridge is unaffected either way, since it gets the CA explicitly.

> **If Use HTTPS is on and no cert exists, the server refuses to start.** It prints one error and turns **Active** back off rather than thrashing the listener every frame.

### macOS

```bash
brew install mkcert
mkcert -install     # installs the root CA; needs your keychain password
```

Toggle **Use HTTPS** and click **Generate**, or do it by hand:

```bash
CAROOT=$(mkcert -CAROOT)
mkcert -cert-file "$CAROOT/localhost.pem" -key-file "$CAROOT/localhost-key.pem" localhost 127.0.0.1
```

Verify:

```bash
curl -X POST https://127.0.0.1:13316/mcp -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'
```

### Windows (PowerShell)

```powershell
winget install FiloSottile.mkcert
mkcert -install     # run as Administrator
```

Toggle **Use HTTPS** and click **Generate**, or by hand:

```powershell
$caroot = mkcert -CAROOT
mkcert -cert-file "$caroot\localhost.pem" -key-file "$caroot\localhost-key.pem" localhost 127.0.0.1
```

Verify:

```powershell
Invoke-RestMethod -Method POST -Uri "https://127.0.0.1:13316/mcp" -ContentType "application/json" -Body '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'
```

With authentication on, this returns `401` with a `WWW-Authenticate: Bearer` header. That is success: it proves TLS and auth enforcement in one shot.

### Connecting over HTTPS

`--transport` names the MCP transport, not the URL scheme. There is no `https` transport; TLS comes from the URL.

```bash
claude mcp remove touchdesigner
claude mcp add --transport http touchdesigner https://127.0.0.1:13316/mcp
```

Two things to expect. If authentication is on you must re-authenticate, because discovery now advertises `https://` and the client's cached identifier no longer matches. And **the generated cert covers `localhost` and `127.0.0.1` only** — reaching the server by any other name, including the IPv6 literal `::1`, fails verification.

## Claude Desktop

Claude Desktop's config file accepts only **stdio** servers, so TDMCP needs the `mcp-remote` stdio-to-HTTP bridge, fetched via `npx`. Node.js 18+ required.

> **Add Custom Connector does not work for local servers.** Connector tool calls are dispatched by Anthropic's backend rather than by the Desktop app, so it cannot resolve a hostname that exists only on your machine. The connector appears to add, then lists zero tools. This is a platform limitation, not a TDMCP bug. Use the bridge below.

Open the config via **Settings → Developer → Edit Config**, which opens the right file every time. By path it is `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows. If the file already has content, add the `mcpServers` key alongside rather than replacing it.

### Plain HTTP (recommended for localhost)

No certs, no HTTPS toggle. `--allow-http` is sanctioned for networks where traffic cannot be intercepted, and loopback qualifies by definition.

```json
{
  "mcpServers": {
    "touchdesigner": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "http://127.0.0.1:13316/mcp", "--allow-http"]
    }
  }
}
```

### HTTPS

For LAN access or environments that forbid `--allow-http`. Complete [HTTPS](#https) first, then point the bridge at `https://` and hand Node the mkcert root CA:

```json
{
  "mcpServers": {
    "touchdesigner": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://127.0.0.1:13316/mcp"],
      "env": { "NODE_EXTRA_CA_CERTS": "/path/to/mkcert-ca/rootCA.pem" }
    }
  }
}
```

`mkcert -CAROOT` prints the directory holding `rootCA.pem`.

Fully quit and restart Claude Desktop after editing, not just close the window.

**With authentication on**, the bridge registers itself, opens your browser, and TouchDesigner shows a consent popup. Click **Allow full control** and the bridge holds the token from there.

### Claude Desktop troubleshooting

**Server disconnects immediately.** Check the log — `~/Library/Logs/Claude/mcp-server-touchdesigner.log` on macOS, `%APPDATA%\Claude\logs\` on Windows. On Windows a spawn error means Desktop could not launch `npx` directly, because it is a `.cmd` shim. Wrap it:

```json
"command": "cmd", "args": ["/c", "npx", "-y", "mcp-remote", "http://127.0.0.1:13316/mcp", "--allow-http"]
```

**`UNABLE_TO_VERIFY_LEAF_CERTIFICATE`.** HTTPS only. Node does not trust the mkcert CA; check `NODE_EXTRA_CA_CERTS` points at the right `rootCA.pem`, or switch to plain HTTP.

**`curl` fails on Windows but the client works.** `SEC_E_UNTRUSTED_ROOT` and `CRYPT_E_NO_REVOCATION_CHECK` are a schannel quirk: it demands a revocation endpoint mkcert certs do not publish. Not a real trust failure. Verify with `Invoke-RestMethod` or `curl --ssl-revoke-best-effort`.

## Multiple instances

MCP configuration is per project directory in Claude Code, so several TD projects can each have their own session.

Give each TDMCP component a unique **Port**, then register each from its own project directory. When you launch the client from that directory it connects to the matching instance.

```bash
cd ~/Projects/ProjectA && claude mcp add --transport http touchdesigner http://127.0.0.1:13316/mcp
cd ~/Projects/ProjectB && claude mcp add --transport http touchdesigner http://127.0.0.1:13317/mcp
```

To run **two components in one project** — say one for Claude Code over HTTP and one for Claude Desktop over HTTPS — give each a unique **Port** *and* a unique **Server Name**. Clients resolve one entry per name, so leaving both as `touchdesigner` means one shadows the other and an agent silently drives the wrong server.

Both components share the same operator tree, so changes made through one are immediately visible to the other.

## OAuth

Optional, enabled by **Ask for authentication**. Authorization Code flow with PKCE (RFC 7636), Dynamic Client Registration (RFC 7591), Authorization Server Metadata (RFC 8414) and Protected Resource Metadata (RFC 9728). Clients discover endpoints via `/.well-known/oauth-authorization-server`; the server also serves `/register`, `/authorize`, `/token`, `/revoke` and `/authorize/status`.

### Where state lives

| State | Location | Survives a TD restart? |
| --- | --- | --- |
| Access tokens, authorization codes | In memory | **No**, by design |
| Client registrations | `.tdmcp/oauth_clients.json`, beside the project | Yes |
| The client's copy of its `client_id` | The client's own config | Yes |

Authorization codes last 300 seconds, access tokens 24 hours. At most 100 client registrations are kept, with LRU eviction that skips any registration backing a live token; if every slot is live, a new registration gets `503`.

`.tdmcp/` is a gitignored sidecar, never part of the `.tox` export, so saving the `.toe` has no effect on it. The path follows `project.folder`, so a Save As carries registrations to the new location.

### Event log

`.tdmcp/oauth_events.log` records `store_load`, `session_reset`, `client_register`, `client_evict`, `authorize_reject`, `consent_allow`, `consent_deny`, `token_issue`, `token_reject` and `auth_rejected`. It rotates once at 256 KB and contains no secret material: tokens, codes and PKCE verifiers never reach it. `client_id` is logged in full, since it is public under PKCE and correlating it with the client's own log is the point.

### Editing the server with a client connected

This affects contributors only. TouchDesigner re-initializes an extension whenever a synced DAT changes, which builds a fresh provider, so **every save of `MCPServerExt.py` or `OAuthProvider.py` discards all tokens and in-flight consents** while registrations reload from disk. A connected client gets a `401` on its next call.

Each occurrence is logged, so it is distinguishable from a real fault:

```
session_reset reason=extension_init
auth_rejected reason=unknown_or_expired_token
```

An `auth_rejected` immediately after a `session_reset` is the expected consequence.

## Adding an agent

Client support is data, not code. Each record in `TDMCP/data/hosts.json` declares where that client discovers skills, how it registers the server, which scopes it supports and the shape of its config file. To add your own, point **Host Override DAT** at a text DAT of JSON:

```json
{
  "mycli": {
    "display": "My CLI",
    "skills": { "strategy": "copy_dir", "project": ".mycli/skills" },
    "mcp": { "add": "mycli mcp add {name} {url}",
             "remove": "mycli mcp remove {name}",
             "scopes": ["project"] }
  }
}
```

`{url}` and `{name}` are filled from the component's live settings; `{scope}` is available for clients whose command takes one. Records are matched by key: a new key is added, an existing key replaced wholesale. A malformed record is skipped and reported in **Skills Status** rather than taking the working records down with it.

Each shipped record carries a `verified` stamp naming the CLI version its claims were last tested against. Client conventions drift fast, so treat `unverified` as a hypothesis.
