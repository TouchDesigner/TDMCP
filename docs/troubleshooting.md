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
- If you changed **Port**, re-register: **Remove MCP** then **Add MCP**. The Claude Desktop plugin cannot be configured and needs the default port, 13316.

## Install Skills reported an error

- **`CERTIFICATE_VERIFY_FAILED`** — TouchDesigner's bundled OpenSSL ships cert paths that do not exist on your machine, so HTTPS from inside TD has no trust anchors. TDMCP falls back to the `certifi` bundle that ships with TD. If you see this, your TD build has no `certifi`; set **Skills Source** to a local TDMCPSkills checkout as a workaround and file an issue.
- **`404 ... no such repo, or it is private`** — check **Skills Repo**. Blank means `TouchDesigner/TDMCPSkills`.
- Anything else — the message carries the underlying reason. Nothing is touched on failure; an existing install stays intact.

## Skills listed twice, or the agent follows old instructions

**Skills Status** says `WARNING: td-* skills in 2 places Claude Code loads`. The agent loads skills from several folders at once, and the same `td-*` skills are in more than one of them, often at different versions. Each skill is then offered twice, and the agent may follow the older one.

Hover **Skills Status** for every folder with its path and version. The usual causes:

- **An old project install on top of the user install.** Keep one. Uninstall with **Uninstall Scope: All agents — this project**, or delete the project's `.claude/skills/td-*` folders.
- **An account plugin.** The TDMCP plugin carries the skills, and a plugin on the account syncs to every Claude client on it, Claude Code included, under `~/.claude/plugins`. So a Claude Code user who has the plugin and also ran **Install Skills** loads both. The component never removes a plugin: keep it and uninstall the component's copy (**Uninstall Scope** at the scope it was installed to), or remove the plugin under **Customize → Plugins** in claude.ai or with `/plugin` in Claude Code.

Only the folders one client loads for this project are compared: the user folder, the project root, and account plugins. Copies in other projects load only in those projects and are not reported.

## Skills installed but the agent ignores them

- Read **Installed Locations** and confirm the directory matches the client you are actually running.
- Scope matters: a project install only applies to sessions started in that directory.
- Restart the agent. Most discover skills at startup.
- For scripted Antigravity, install at **user** scope — see [Skills](component.md#skills).

## `UNKNOWN_CERTIFICATE_VERIFICATION_ERROR`

Claude Code shows `touchdesigner (UNKNOWN_CERTIFICATE_VERIFICATION_ERROR): "unknown certificate verification error"`, or Node reports `UNABLE_TO_VERIFY_LEAF_SIGNATURE`. **Use HTTPS** is on, and the client does not trust the certificate, even though `curl` and the browser do.

The cause: the client runs on Node.js, which ignores the operating system's trust store, and `mkcert -install` only wrote to that store. Two fixes:

- **Turn Use HTTPS off.** You do not need it on localhost. Then follow [Recovering from Use HTTPS](#recovering-from-use-https), because the client is still registered with the `https://` URL.
- **Or keep HTTPS and tell Node about the root CA:** start the client with `NODE_EXTRA_CA_CERTS="$(mkcert -CAROOT)/rootCA.pem"` set. Details and the Windows form in [Connecting over HTTPS](setup-advanced.md#connecting-over-https).

## Recovering from Use HTTPS

Turning **Use HTTPS** on or off changes the server's URL scheme, and clients connect with the URL they registered. A client registered with the other scheme does not recover on its own. The component prints the new URL in the textport when you flip it.

1. **In TouchDesigner**, set **Use HTTPS** the way you want it. Off is right for nearly everyone. **Add MCP** follows the toggle.
2. **Check what each client registered:** `claude mcp get touchdesigner` shows the URL.
   - **It still has the right scheme** (you flipped the toggle and flipped it back, or never re-registered): its next call reconnects on its own, with the same sign-in. If it still reports an error, reconnect it with `/mcp` in Claude Code.
   - **It has the other scheme:** remove it and add it again with the command now in **Add MCP**, then restart its session:
     ```bash
     claude mcp remove touchdesigner
     claude mcp add --transport http touchdesigner http://127.0.0.1:13316/mcp
     ```
     Prefer copying **Add MCP**: it carries your scope, port and **Server Name**. For the Code tab, use its own command from **Install For: Claude Desktop (Code tab)**.
   - **Claude Desktop plugin:** it speaks plain HTTP only. With Use HTTPS off again, its next call reconnects on its own; if it stays silent, turn the plugin off and on under **Customize → Plugins**.
3. **Sign in again if asked.** With **Ask for authentication** on, a client that tried to sign in while the scheme was different may hold stale details. Allow it in the consent panel. If it keeps being refused, press **Revoke Access** on the Storage page and reconnect it.

## OAuth: 401, or "does not match expected", after a restart or a URL change

Clear the client's stored authentication and reconnect. In Claude Code: `/mcp` → `touchdesigner` → *Clear authentication* → *Reconnect*, then approve the TouchDesigner popup. Other clients: remove and re-add the server.

Tokens last 24 hours and survive a TD restart, so a bare `401` after a restart means the token had already expired or registrations were cleared. The server also advertises itself exactly as you addressed it, so switching between `localhost` and `127.0.0.1` invalidates the client's cached identifier.

Restarting TouchDesigner does not help. The stale state is in the client.

## Stuck in a re-authentication loop

The consent page opens but never completes. The client is replaying a `client_id` this server does not have, and re-authenticating preserves registrations while clearing only tokens, so it fails identically every time.

Confirm in the per-port log, `logs/tdmcp_13316.log` in the TDMCP state folder (substitute your port; the **Storage** page shows the exact path and opens the folder): `grep authorize_reject` on macOS, `Select-String authorize_reject` on Windows. `reason=unknown_client_id` is this failure. Fix by clearing the *registration*: `claude mcp logout touchdesigner`, or remove and re-add for other clients. For the Claude Desktop bridge, see *Authentication error after Clear Registrations* under Claude Desktop problems below.

## Claude Desktop problems

- **Upload plugin says the archive must contain `.claude-plugin/plugin.json`.** You gave it `TDMCP.mcpb`, or a zip of it. That page takes `Derivative-TDMCP.plugin` from the release, which wraps the bundle in a plugin manifest.
- **Nothing under Connectors.** That page lists remote connectors. The plugin's server is on the plugin's own **Connectors** tab under **Customize → Plugins**, marked **Runs in each session**, and shows as **Tdmcp** in a session's connector list.
- **Tools missing after uploading the plugin.** A session starts the bridge when it opens, and TouchDesigner must be Active by then. Make it Active and start a new session; or turn the plugin off and on under **Customize → Plugins**, or fully quit and relaunch Desktop.
- **You uploaded a newer plugin, but a session still reports the old bridge version.** A session keeps the bridge process it started with. Start a new session.
- **Add Custom Connector shows no tools.** It cannot reach a local server at all: connector calls come from Anthropic's cloud. Use `Derivative-TDMCP.plugin`, or the manual bridge in [docs/setup-advanced.md#claude-desktop](setup-advanced.md#claude-desktop).
- **Authentication error after Clear Registrations.** The bridge caches its client id under `~/.mcp-auth` and keeps presenting it after the server has forgotten it, so every connect is refused. Delete that folder, then turn the plugin off and on; it registers fresh and the consent panel appears once. To cut off access without this, use **Revoke Access** instead: it keeps the registration, so the bridge simply signs in again. See [Revoke Access or Clear Registrations](setup-advanced.md#ending-access-revoke-access-or-clear-registrations).
- **You changed Port.** The plugin cannot be configured and always uses 13316. Put the component back on 13316, or use the manual config.
- **You turned the plugin off, but sessions still have TouchDesigner tools.** Desktop applies the **Disable plugin** toggle to local servers only when it next refreshes its plugin state or restarts, which can be an hour later; until then a running bridge keeps its access and new sessions get one too. To cut access now, press **Revoke Access** on the component's Storage page, or turn **Active** off. Removing the plugin and relaunching Desktop also ends it.
- **You turned on Use HTTPS.** The bundle speaks plain HTTP only, which is safe on loopback, and goes silent when the server switches to TLS. Turn Use HTTPS off; the bundle needs nothing from mkcert. Ask for authentication stays on, as shipped.
- **Desktop says it can't see `view_operator` images, only a file path.** Turn on **Inline Images (base64)** on the Tune page. Desktop cannot read the cache folder the path points to. More in [Tuning for Claude Desktop](setup-advanced.md#tuning-for-claude-desktop).
- Desktop's MCP logs are under **Settings → Developer**.
