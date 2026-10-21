# Troubleshooting

Start with the first section; it covers most reports. Every section assumes
**Active** is on and the client was restarted after registering the server.

## The agent has no TouchDesigner tools

1. Check **Active** is on.
2. Test the server directly. A JSON reply means it is up; `401` means it is up with authentication on, which is also success:
   ```bash
   curl -X POST http://127.0.0.1:13316/mcp -H "Content-Type: application/json" \
     -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'
   ```
3. Confirm the port in **Add MCP** matches what your client registered.
4. Restart the client. Most only read MCP config at startup.

## Connection refused

- **Use `127.0.0.1`, never `localhost`.** The socket is IPv4-only. `localhost` resolves to `::1` first on many systems, and a client that does not retry the next address reports a refusal.
- Check nothing else holds the port.
- If you changed **Port**, re-register: **Remove MCP** then **Add MCP**. For Claude Desktop, change it under **Settings → Extensions → TouchDesigner**.

## Install Skills reported an error

- **`CERTIFICATE_VERIFY_FAILED`** — TouchDesigner's bundled OpenSSL ships cert paths that do not exist on your machine, so HTTPS from inside TD has no trust anchors. TDMCP falls back to the `certifi` bundle that ships with TD. If you see this, your TD build has no `certifi`; set **Skills Source** to a local TDMCPSkills checkout as a workaround and file an issue.
- **`404 ... no such repo, or it is private`** — check **Skills Repo**. Blank means `TouchDesigner/TDMCPSkills`.
- Anything else — the message carries the underlying reason. Nothing is touched on failure; an existing install stays intact.

## Skills installed but the agent ignores them

- Read **Installed Locations** and confirm the directory matches the client you are actually running.
- Scope matters: a project install only applies to sessions started in that directory.
- Restart the agent. Most discover skills at startup.
- For scripted Antigravity, install at **user** scope — see [Skills](component.md#skills).

## OAuth: 401, or "does not match expected", after a restart or a URL change

Clear the client's stored authentication and reconnect. In Claude Code: `/mcp` → `touchdesigner` → *Clear authentication* → *Reconnect*, then approve the TouchDesigner popup. Other clients: remove and re-add the server.

Tokens last 24 hours and survive a TD restart, so a bare `401` after a restart means the token had already expired or registrations were cleared. The server also advertises itself exactly as you addressed it, so switching between `localhost` and `127.0.0.1` invalidates the client's cached identifier.

Restarting TouchDesigner does not help. The stale state is in the client.

## Stuck in a re-authentication loop

The consent page opens but never completes. The client is replaying a `client_id` this server does not have, and re-authenticating preserves registrations while clearing only tokens, so it fails identically every time.

Confirm in the per-port log, `logs/tdmcp_13316.log` in the TDMCP state folder (substitute your port; the **Storage** page shows the exact path and opens the folder): `grep authorize_reject` on macOS, `Select-String authorize_reject` on Windows. `reason=unknown_client_id` is this failure. Fix by clearing the *registration*: `claude mcp logout touchdesigner`, or remove and re-add for other clients.

## Claude Desktop problems

- **Dropped the file and nothing happened.** Drop it on the **Settings → Extensions** page, not the chat window. After an uninstall that page stops accepting drops until you leave it and come back, or relaunch Desktop. Desktop also silently ignores a bundle it has installed before, so make sure you are dragging the current file.
- **Tools missing after installing the bundle.** TouchDesigner must be Active before Desktop starts the bridge. Turn the extension off and on under **Settings → Extensions**, or fully quit and relaunch Desktop.
- **Add Custom Connector shows no tools.** It cannot reach a local server at all: connector calls come from Anthropic's cloud. Use the `TDMCP.mcpb` bundle, or the manual bridge in [docs/setup-advanced.md#claude-desktop](setup-advanced.md#claude-desktop).
- **Authentication error after Clear Registrations.** The bridge caches its client id under `~/.mcp-auth` and keeps presenting it after the server has forgotten it, so every connect is refused. Delete that folder, then turn the extension off and on; it registers fresh and the consent panel appears once.
- **You changed Port.** The bundle keeps its own copy: **Settings → Extensions → TouchDesigner → Configure**.
- **You turned on Use HTTPS.** The bundle speaks plain HTTP only, which is safe on loopback, and goes silent when the server switches to TLS. Turn Use HTTPS off; the bundle needs nothing from mkcert. Ask for authentication stays on, as shipped.
- **Desktop says it can't see `view_operator` images, only a file path.** Turn on **Inline Images (base64)** on the Tune page. Desktop cannot read the cache folder the path points to. More in [Tuning for Claude Desktop](setup-advanced.md#tuning-for-claude-desktop).
- Desktop's MCP logs are under **Settings → Developer**.
