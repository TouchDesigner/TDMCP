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
- **The plugin and a copy on disk.** The TDMCP plugin carries the skills, and a plugin on the account syncs to every Claude client on it, Claude Code included, under `~/.claude/plugins`. A copy that **Install Skills** made before the plugin was uploaded then loads beside it. Keep the plugin and press **Uninstall Skills** at the scope the copy was installed to. From 1.1.58, **Install Skills** makes no second copy while the plugin provides the skills: **Skills Status** reads `Provided by the account plugin`, and installing a separate copy takes a **Skills Source** or **Skills Version**. The component never removes a plugin; do that under **Customize → Plugins** in claude.ai or with `/plugin` in Claude Code.
- **Two versions shown for the plugin.** A plugin from 1.1.58 on is listed with its skills version, such as `v1.4.2 (account plugin)`. An older one is listed as `TDMCP 1.1.57 plugin`, because it only records the TDMCP version it shipped with.

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

## Claude Desktop

Sign-in that never finishes, a server Desktop shows as failed, missing tools
and the other Desktop problems are covered on the
[Claude Desktop](claude-desktop.md#troubleshooting) page.
