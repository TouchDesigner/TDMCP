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

Headless also does not mount workspace skills. Install at user scope — see [Skills](component.md#skills).

## HTTPS

**You do not need HTTPS on this machine.** With the default **Server Address Scope** (Localhost only), every request travels over loopback, which nothing can intercept, so TLS adds nothing. Plain HTTP on `127.0.0.1` is the recommended setup: Claude Code connects to it directly, the Claude Desktop bundle speaks only HTTP, and the bridge accepts it with `--allow-http`. HTTPS is required only for **All interfaces**, where requests cross the network. Set it up for that, or when your environment forbids `--allow-http`.

Turning **Use HTTPS** on changes the server's URL from `http://` to `https://`. Every client registered with the old URL stops connecting until you re-register it, and the Claude Desktop bundle goes silent. Turned on by mistake? See [Recovering from Use HTTPS](troubleshooting.md#recovering-from-use-https).

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

**Node-based clients must be told to trust the mkcert root.** Claude Code, the Code tab bridge and `mcp-remote` run on Node.js, which checks certificates against its own bundled list of root CAs and ignores the operating system's. `mkcert -install` puts its root CA in the OS store (the macOS Keychain, the Windows certificate store), so `curl` and your browser trust the server while Node still refuses it, with `UNKNOWN_CERTIFICATE_VERIFICATION_ERROR` in Claude Code or `UNABLE_TO_VERIFY_LEAF_SIGNATURE` from Node itself. Every check that passes is reading a different trust store from the one that fails. Set `NODE_EXTRA_CA_CERTS` to the root CA in the environment that starts the client:

```bash
# macOS / Linux: in the shell you start claude from, or your shell profile
export NODE_EXTRA_CA_CERTS="$(mkcert -CAROOT)/rootCA.pem"
claude
```

```powershell
# Windows PowerShell
$env:NODE_EXTRA_CA_CERTS = "$(mkcert -CAROOT)\rootCA.pem"
claude
```

A registration cannot carry it for Claude Code connecting directly: Claude Code makes the TLS connection itself, so the variable must be set when it starts. For a bridge it can, because the bridge is its own process: with Use HTTPS on, **Install For: Claude Desktop (Code tab)** generates `claude mcp add … touchdesigner -e "NODE_EXTRA_CA_CERTS=<root CA>" -- npx …` with the path filled in. `NODE_OPTIONS=--use-system-ca`, which asks Node to read the OS store instead, did not accept the mkcert root in testing on macOS with Node 26, so it is not recommended here.

## Claude Desktop

Claude Desktop runs local MCP servers over **stdio**, so TDMCP reaches it through a bridge, `TDMCP.mcpb`, which the release ships inside a Claude plugin, `Derivative-Official-TDMCP-Plugin.zip`, together with the `td-*` skills. One install covers chat, Cowork and Code sessions on your computer, and the skills load in all three with nothing to install. It does not appear under **Connectors**: that page lists remote servers, which TDMCP is not.

### Install the plugin

Download `Derivative-Official-TDMCP-Plugin.zip` from the same [release](https://github.com/TouchDesigner/TDMCP/releases) as the `.tox`. In Claude Desktop open **Customize → Plugins → Add → Upload plugin** and drop the file in. The preview shows the plugin's name, `touchdesigner`, and its description; click **Upload**, then accept the prompt that the plugin includes a local MCP server, `tdmcp`, which runs on your computer. It appears under **Your plugins** as **TouchDesigner**, with `TDMCP.mcpb` on its **Connectors** tab marked **Runs in each session**. Any chat, Cowork or Code session started after that runs the bridge, provided TouchDesigner is running with **Active** on; a session's connector list shows the server as **Tdmcp**.

The plugin uses the bundle's default port, 13316, and Desktop offers no settings for a plugin-bundled server. A component whose **Port** was changed cannot be reached from Desktop: put it back, or use the [manual config](#manual-config) below.

**Updating.** Upload the newer `Derivative-Official-TDMCP-Plugin.zip` the same way; Desktop replaces the installed plugin in place and the plugins page shows the new version. A session that is already open keeps the bridge it started with, so start a new session to pick up the new one. The plugin's version is its TDMCP version; use it with the `.tox` from the same release. Plugins are saved to your account, so the same upload also reaches Claude Code signed in as you, as a synced plugin.

The bundle carries its own copy of the `mcp-remote` bridge and runs on the Node.js inside Claude Desktop. Nothing to install, no config file. The host is fixed to `127.0.0.1`.

**HTTP, HTTPS and authentication with the bundle.** The bundle talks plain HTTP on loopback, which nothing on the machine can intercept, so it needs no certificate and nothing from mkcert. Leave **Use HTTPS** off on the component: the bundle only speaks HTTP and goes silent if the server switches to TLS. **Ask for authentication** is on as shipped and works as it does for any client: the first connection opens a browser tab, TouchDesigner raises its consent panel under the mouse, and the bridge keeps its registration under `~/.mcp-auth`; the access token lives in the TDMCP state folder for 24 hours, so a TouchDesigner restart within that window reconnects without asking. To make the server forget the bridge, use **Clear Registrations** on the component's Storage page, delete the bridge's own cache (`~/.mcp-auth`, where it keeps the client id the server just forgot; the bridge does not re-register on its own, it keeps presenting the old id and is refused as `unknown_client_id`), then turn the plugin off and on in Desktop.

### Manual config

For environments that cannot install plugins, or a component on a port other than 13316. Needs Node.js 18+ on your machine, since the bridge is fetched via `npx`.

> **Add Custom Connector does not work for local servers.** Connector tool calls are dispatched by Anthropic's backend rather than by the Desktop app, so it cannot resolve a hostname that exists only on your machine. The connector appears to add, then lists zero tools. This is a platform limitation, not a TDMCP bug. Use the bridge below.

Open the config via **Settings → Developer → Edit Config**, which opens the right file every time. By path it is `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows. If the file already has content, add the `mcpServers` key alongside rather than replacing it.

### Plain HTTP (recommended for localhost)

No certs, no HTTPS toggle. `--allow-http` is sanctioned for networks where traffic cannot be intercepted, and loopback qualifies by definition.

```json
{
  "mcpServers": {
    "touchdesigner": {
      "command": "npx",
      "args": ["-y", "mcp-remote@0.15.1", "http://127.0.0.1:13316/mcp", "--allow-http"]
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
      "args": ["-y", "mcp-remote@0.15.1", "https://127.0.0.1:13316/mcp"],
      "env": { "NODE_EXTRA_CA_CERTS": "/path/to/mkcert-ca/rootCA.pem" }
    }
  }
}
```

`mkcert -CAROOT` prints the directory holding `rootCA.pem`.

Fully quit and restart Claude Desktop after editing, not just close the window.

**With authentication on**, the bridge registers itself, opens your browser, and TouchDesigner shows a consent popup. Click **Allow full control** and the bridge holds the token from there.

### Claude Desktop Code tab

The Code tab runs Claude Code inside Desktop, and it reads the same registrations as the `claude` CLI. The plain `claude mcp add --transport http …` command that **Install For: Claude Code** generates registers fine there but cannot sign in: the Code tab's HTTP client refuses any sign-in page that is not `https`, and fails with `Refused to open sign-in URL for touchdesigner: must be https`. The terminal CLI is not affected.

Register through the `mcp-remote` bridge instead, the same one the bundle uses. It opens the sign-in page itself, so the consent panel appears and the connection completes. **Install For: Claude Desktop (Code tab)** generates this command for your platform, port and scope; by hand, run it from the project folder:

```bash
# macOS / Linux
claude mcp add --scope local touchdesigner -- npx -y mcp-remote@0.15.1 http://127.0.0.1:13316/mcp --allow-http

# Windows (npx is a .cmd shim, so wrap it)
claude mcp add --scope local touchdesigner -- cmd /c npx -y mcp-remote@0.15.1 http://127.0.0.1:13316/mcp --allow-http
```

Needs Node.js 18+. `--allow-http` is safe here for the reason given under [Plain HTTP](#plain-http-recommended-for-localhost): the URL is loopback. Never use it with any other host. The version is pinned so `npx` runs the same `mcp-remote` the bundle ships instead of whatever is newest. If you already added the plain HTTP registration, remove it first with `claude mcp remove touchdesigner`.

### Tuning for Claude Desktop

The component ships tuned for Claude Code, which reads files from disk. Desktop cannot, so a few settings work better changed.

**Turn on Inline Images (base64)** on the **Tune** page. With it off, `view_operator` writes the capture to `cache/<port>/` and returns only the file path. Claude Code opens that file; Desktop has no file access, so it never sees the image and falls back to reading pixel values with `inspect_values`. With Inline Images on, the PNG comes back inside the tool result and Desktop sees it directly. Each capture costs more context than a path, so keep `view_operator` at its default `tiny` size unless detail matters.

### Claude Desktop troubleshooting

**Server disconnects immediately.** Check the log — `~/Library/Logs/Claude/mcp-server-touchdesigner.log` on macOS, `%LOCALAPPDATA%\Claude\logs\` on Windows (older installs wrote to `%APPDATA%\Claude\logs\`; **Settings → Developer** opens the right folder either way). The file is named after the server: `mcp-server-TouchDesigner.log` for the bundle, `mcp-server-touchdesigner.log` for a manual config. On Windows a spawn error means Desktop could not launch `npx` directly, because it is a `.cmd` shim. Wrap it:

```json
"command": "cmd", "args": ["/c", "npx", "-y", "mcp-remote@0.15.1", "http://127.0.0.1:13316/mcp", "--allow-http"]
```

**`UNABLE_TO_VERIFY_LEAF_CERTIFICATE`.** HTTPS only. Node does not trust the mkcert CA; check `NODE_EXTRA_CA_CERTS` points at the right `rootCA.pem`, or switch to plain HTTP.

**`curl` fails on Windows but the client works.** `SEC_E_UNTRUSTED_ROOT` and `CRYPT_E_NO_REVOCATION_CHECK` are a schannel quirk: it demands a revocation endpoint mkcert certs do not publish. Not a real trust failure. Verify with `Invoke-RestMethod` or `curl --ssl-revoke-best-effort`.

## Several clients on one instance

It is common to have Claude Desktop (through `TDMCP.mcpb`) and Claude Code, or another agent, connected to the same TouchDesigner at once. That works, and needs no extra setup: each client keeps its own registration, so both can be called `touchdesigner`.

**What they share:** the TouchDesigner session itself. Both agents see and change the same network, the same parameters and the same undo history. Neither knows the other is there.

**What they do not share:**

- **Sign-in.** Each client registers and is allowed separately; the consent panel appears once for each.
- **Skills.** Each client reads its own skills folders. Claude Code and the Desktop Code tab share Claude Code's folders; Claude Desktop chat loads skills only from plugins, which is why the TDMCP plugin carries them. A plugin on the account also syncs to Claude Code, so a Claude Code user with the plugin needs no **Install Skills**. **Skills Status** warns if one client loads two copies.
- **Conversation.** Each agent has only its own context, and does not see what the other just built.

**The risk is two agents editing the same thing.** One agent can rename, rewire or delete what the other is working on, and each will report success for its own step. Keep them apart:

- Give each agent its own COMP to work in, and say so in your prompt.
- Or let one agent build while the other only reads: reviewing, inspecting values, viewing operators.
- Watch **Connected Clients** on the component's Storage page. It lists every client that sent a request in the last 10 minutes, with the name and version it reports, and shows the Desktop bundle as `via TDMCP.mcpb <version>`.
- To stop all of them at once, press **Revoke Access** on the same page. Each must sign in and be allowed again.

**Separate Server Names or ports are for separate TouchDesigner instances, not separate clients.** Two clients on one instance use the same port and the same name. Two instances need a different **Port** and **Server Name** each, as below.

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

On by default, controlled by **Ask for authentication**. Authorization Code flow with PKCE (RFC 7636), Dynamic Client Registration (RFC 7591), Authorization Server Metadata (RFC 8414) and Protected Resource Metadata (RFC 9728). Clients discover endpoints via `/.well-known/oauth-authorization-server`; the server also serves `/register`, `/authorize`, `/token`, `/revoke` and `/authorize/status`.

### Where state lives

| State | Location | Survives a TD restart? |
| --- | --- | --- |
| Access tokens | `oauth_tokens_<port>.json` in the TDMCP state folder, user-only permissions | Yes, for their 24-hour lifetime |
| Authorization codes, PKCE material, remembered consent | In memory | **No**, by design |
| Client registrations | `oauth_clients_<port>.json` in the TDMCP state folder | Yes |
| Image captures (`view_operator`, Inline Images off) | `cache/<port>/` in the TDMCP state folder | No, cleared at server start |
| The client's copy of its `client_id` | The client's own config | Yes |

Authorization codes last 300 seconds, access tokens 24 hours. At most 100 client registrations are kept, with LRU eviction that skips any registration backing a live token; because tokens persist, that guard holds from the first second after a restart too. If every slot is live, a new registration gets `503`.

**The consent panel is non-modal.** It is a floating window owned by the component (`/TDMCP/dialog`), with the request's facts, a countdown, and two buttons: **Deny** and **Allow full control**. Because it is not a modal dialog, the server keeps serving every other client while it is open, and a request nobody answers expires after five minutes instead of hanging the server (#59). Requests from the same application that are waiting at the same time, meaning the same client or the same name and callback registered separately, are shown as one entry and answered with one click; Claude Desktop starts the plugin's server twice when it connects, so this is what keeps that to a single Allow. If the panel cannot open, the component falls back to TouchDesigner's modal message box.

**Consent is remembered per client for the session.** The panel appears the first time a client asks; later authorizations from the same registration are approved without it, logged as `consent_reuse`, until TouchDesigner restarts, **Revoke Access** or **Clear Registrations** runs. A Deny forgets the client.

The TDMCP state folder sits beside TouchDesigner's own preferences folder, so it survives TD upgrades and is never cloud-synced: `%LOCALAPPDATA%\Derivative\TDMCP` on Windows, `~/Library/Application Support/Derivative/TDMCP` on macOS. It is keyed by port because clients identify a server by its URL: two components on different ports keep separate files, and every project served on one port shares them (registration is identity; the consent popup still gates every token). Changing **Port** starts a fresh file for the new port. Nothing is written beside the project, so an unsaved project no longer leaves a hidden `.tdmcp/` where TouchDesigner was launched from. A `.tdmcp/oauth_clients.json` left beside a project by 1.1.55 is imported once on the next start, logged as `store_import`, and then ignored; delete the old folder when convenient. The component's **Storage** page shows the resolved paths for the running port, opens the folder, and clears the registrations, logs or cache.

### Ending access: Revoke Access or Clear Registrations

Both buttons are on the component's **Storage** page, and both end every session on this port at once. The difference is whether the server still knows the clients afterwards.

| | **Revoke Access** | **Clear Registrations** |
| --- | --- | --- |
| Access tokens and `oauth_tokens_<port>.json` | Removed | Removed |
| Unused authorization codes | Removed | Removed |
| Remembered consent | Forgotten | Forgotten |
| Client registrations and `oauth_clients_<port>.json` | **Kept** | **Removed**; the file stays, emptied |
| What a connected client does next | Its next request is refused; it signs in again under its existing client id and the consent panel asks you again | Its next request is refused, and it must register as a new client, then sign in, and the consent panel asks you again |
| Claude Code and other HTTP clients | Sign in again on their next request | Must register again; if one keeps failing, reconnect it (`/mcp` in Claude Code) |
| Claude Desktop bridge (`TDMCP.mcpb`, `mcp-remote`) | Signs in again on its next request, under the client id it cached | **Stays refused** as `unknown_client_id`: it caches its client id in `~/.mcp-auth` and keeps presenting it. Delete that folder and turn the plugin off and on |
| Logged as | `access_revoke tokens=<n> clients=<n>` | `store_clear clients=<n>` |

**Use Revoke Access** to cut off every connected agent: after you stop working, when you are not sure what is connected, or before handing the machine to someone else. Nothing reconnects without you clicking **Allow** in the consent panel again, and nothing needs repair afterwards.

**Use Clear Registrations** to start from nothing: to drop old clients from the list, after changing which clients you use, or when a registration itself is bad. Expect to clear the Desktop bridge's cache as above.

Neither turns authentication off or blocks a client from asking again; they end what is already allowed. To stop new connections, turn **Active** off, or leave **Ask for authentication** on and deny the request. Do not rely on the plugin's **Disable plugin** toggle in Claude Desktop for this: Desktop applies it to a local server only when it next refreshes its plugin state or restarts, and a running bridge keeps its token until then. Press them in TouchDesigner. Ordinary tools cannot press them, since tools cannot change the TDMCP component; only `execute_code` reaches it (see [Security](security.md#security)).

### Event log

`logs/tdmcp_<port>.log` in the state folder records `store_import`, `store_load`, `store_clear`, `access_revoke`, `token_load`, `log_clear`, `session_reset`, `client_register`, `client_evict`, `authorize_reject`, `consent_allow`, `consent_reuse`, `consent_deny`, `token_issue`, `token_reject` and `auth_rejected`. It rotates once at 256 KB and contains no secret material: tokens, codes and PKCE verifiers never reach it. `client_id` is logged in full, since it is public under PKCE and correlating it with the client's own log is the point.

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
