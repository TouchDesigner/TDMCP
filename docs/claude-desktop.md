# Claude Desktop

TDMCP connects to Claude Desktop through one plugin, `derivative_tdmcp.plugin`,
from the [release page](https://github.com/TouchDesigner/TDMCP/releases). It
carries the bridge that links Desktop to the component in TouchDesigner, and
the `td-*` skills that teach Claude how TouchDesigner networks are built.

## What you need

- **[TouchDesigner](https://derivative.ca/download) 2025.33230 or later**, with `TDMCP.tox` in your project and
  **Active** on. The plugin always connects on port 13316, the component's
  default.
- **Claude Desktop** on macOS or Windows, signed in to a **paid plan**: Pro,
  Max, Ultra, Team or Enterprise.
- **Cowork.** TouchDesigner's tools load in **Cowork** sessions and in the
  **Code** tab. Plain **Chat** does not load a plugin's local server, so a Chat
  conversation has no TouchDesigner tools. Pick **Cowork** under the prompt box
  when you start a conversation. Anthropic describes Cowork in
  [Get started with Cowork](https://support.claude.com/en/articles/13345190-get-started-with-cowork).

Nothing else to install: the plugin carries its own copy of the bridge and runs
on the Node.js inside Claude Desktop.

## Quick start

1. Open your project in TouchDesigner and drag `TDMCP.tox` into the network.
   Click the new `tdmcp` node, press **P** if no parameter window is showing,
   and on its **MCP** page switch **Active** on.
2. In Claude Desktop open **Customize → Plugins → Add → Upload plugin** and
   drop `derivative_tdmcp.plugin` in. Accept the prompt that it runs a local
   server on your computer.
3. Quit Claude Desktop fully (File → Exit on Windows, Claude → Quit on macOS)
   and open it again.
4. A browser tab opens, and TouchDesigner shows a consent panel naming the
   client. Press **Allow full control**, and leave the tab open until it
   returns you to the app.
5. Start a **Cowork** session and ask: *What TouchDesigner project am I
   connected to?*

## Install the plugin

Download `derivative_tdmcp.plugin` from the same [release](https://github.com/TouchDesigner/TDMCP/releases) as the `.tox`. In Claude Desktop open **Customize → Plugins → Add → Upload plugin** and drop the file in. The preview shows the plugin's name, `touchdesigner`, and its description; click **Upload**, then accept the prompt that the plugin includes a local MCP server, `tdmcp`, which runs on your computer. It appears under **Your plugins** as **TouchDesigner**, with `tdmcp.mcpb` on its **Connectors** tab marked **Runs in each session**. Any Cowork or Code session started after that runs the bridge, provided TouchDesigner is running with **Active** on; a session's connector list shows the server as **Tdmcp**.

**Start TouchDesigner first.** Desktop starts the plugin's bridge when it launches and when a session opens. If the component is not **Active** on port 13316 at that moment, the bridge stops at once and Desktop retries later, on its own schedule: measured once, the next attempt came 39 seconds after TouchDesigner was ready, after three minutes of nothing. Start a new session, or quit and reopen Desktop, to connect sooner. After **Allow full control**, the tools reach the session in about ten seconds.

The plugin uses the bundle's default port, 13316, and Desktop offers no settings for a plugin-bundled server. A component whose **Port** was changed cannot be reached from Desktop: put it back, or use the [manual config](#manual-config) below.

**Updating.** Upload the newer `derivative_tdmcp.plugin` the same way; Desktop replaces the installed plugin in place and the plugins page shows the new version. A session that is already open keeps the bridge it started with, so start a new session to pick up the new one. The plugin's version is its TDMCP version; use it with the `.tox` from the same release. Plugins are saved to your account, so the same upload also reaches Claude Code signed in as you, as a synced plugin.

The bundle carries its own copy of the `mcp-remote` bridge and runs on the Node.js inside Claude Desktop. Nothing to install, no config file. The host is fixed to `127.0.0.1`.

**HTTP, HTTPS and authentication with the bundle.** The bundle talks plain HTTP on loopback, which nothing on the machine can intercept, so it needs no certificate and nothing from mkcert. Leave **Use HTTPS** off on the component: the bundle only speaks HTTP and goes silent if the server switches to TLS. **Ask for authentication** is on as shipped and works as it does for any client: the first connection opens a browser tab, TouchDesigner raises its consent panel in the middle of the screen the mouse is on, and the bridge keeps its registration under `~/.mcp-auth`; the access token lives in the TDMCP state folder for 24 hours, so a TouchDesigner restart within that window reconnects without asking. To make the server forget the bridge, use **Clear Registrations** on the component's Storage page, delete the bridge's own cache (`~/.mcp-auth`, where it keeps the client id the server just forgot; the bridge does not re-register on its own, it keeps presenting the old id and is refused as `unknown_client_id`), then turn the plugin off and on in Desktop.

## Claude Code and the plugin

A plugin uploaded in Claude Desktop is saved to your claude.ai account, and Claude Code signed in to the same account syncs it at session start, into `~/.claude/plugins/synced`. Desktop is needed to upload the plugin, not to run it. In Claude Code:

- The server is listed as `plugin:touchdesigner:tdmcp` and the skills as `touchdesigner:td-*`.
- Nothing is installed per project, and **Install Skills** has nothing to do: **Skills Status** reads `Provided by the account plugin`.
- **Do not also register the server with `claude mcp add` on port 13316.** Both connections would reach the same component and every tool would be listed twice. A separate registration is for a second component on another port.
- After uploading a newer plugin, start a new Claude Code session. If the skills are missing in that first session, run `/reload-skills` or start once more: the sync can finish just after the session has read its skills.

Use **Add MCP** and **Install Skills** for Claude Code only when there is no claude.ai account to sync from (an API key, Amazon Bedrock or Google Vertex AI login), when the component is on a port other than 13316, or when a project must pin an older skills version; set **Skills Version** for that last case.

## Manual config

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
      "args": ["-y", "mcp-remote@0.16.6", "http://127.0.0.1:13316/mcp", "--allow-http"]
    }
  }
}
```

### HTTPS

For LAN access or environments that forbid `--allow-http`. Complete [HTTPS](setup-advanced.md#https) first, then point the bridge at `https://` and hand Node the mkcert root CA:

```json
{
  "mcpServers": {
    "touchdesigner": {
      "command": "npx",
      "args": ["-y", "mcp-remote@0.16.6", "https://127.0.0.1:13316/mcp"],
      "env": { "NODE_EXTRA_CA_CERTS": "/path/to/mkcert-ca/rootCA.pem" }
    }
  }
}
```

`mkcert -CAROOT` prints the directory holding `rootCA.pem`.

Fully quit and restart Claude Desktop after editing, not just close the window.

**With authentication on**, the bridge registers itself, opens your browser, and TouchDesigner shows a consent panel naming the bridge. If you allow it, the bridge holds the token from there.

## Claude Desktop Code tab

The Code tab runs Claude Code inside Desktop, and it reads the same registrations as the `claude` CLI. The plain `claude mcp add --transport http …` command that **Install For: Claude Code** generates registers fine there but cannot sign in: the Code tab's HTTP client refuses any sign-in page that is not `https`, and fails with `Refused to open sign-in URL for touchdesigner: must be https`. The terminal CLI is not affected.

Register through the `mcp-remote` bridge instead, the same one the bundle uses. It opens the sign-in page itself, so the consent panel appears and the connection completes. **Install For: Claude Desktop (Code tab)** generates this command for your platform, port and scope; by hand, run it from the project folder:

```bash
# macOS / Linux
claude mcp add --scope local touchdesigner -- npx -y mcp-remote@0.16.6 http://127.0.0.1:13316/mcp --allow-http

# Windows (npx is a .cmd shim, so wrap it)
claude mcp add --scope local touchdesigner -- cmd /c npx -y mcp-remote@0.16.6 http://127.0.0.1:13316/mcp --allow-http
```

Needs Node.js 18+. `--allow-http` is safe here for the reason given under [Plain HTTP](#plain-http-recommended-for-localhost): the URL is loopback. Never use it with any other host. The version is pinned so `npx` runs the same `mcp-remote` the bundle ships instead of whatever is newest. If you already added the plain HTTP registration, remove it first with `claude mcp remove touchdesigner`.

## Images

**Images need no setting.** `view_operator` picks how to return a capture by who is asking:

| Caller | Gets |
|---|---|
| Desktop Cowork session (through the plugin's bridge) | The image, inside the reply |
| Claude Code, by any route, including a Desktop Code session | The path of the PNG in `cache/<port>/`, which it opens when it needs to look |
| Any other client | The path, with a one-line `note` naming **Inline Images** |

**Inline Images (base64)** on the **Tune** page turns inline images on for every caller. Turn it on for another agent that cannot open local files; its note says so.

Why not inline for everyone: every inline capture lands in the conversation whether or not the agent needed to look, where a path costs a line until the file is opened. That is small at the default `tiny` size and grows with `low`, `high` and multi-frame grids. Claude Code reads files, so it pays only for the images it opens. Keep `view_operator` at `tiny` unless detail matters.

The caller is known from the name its client gave when it connected, as shown under **Connected Clients**. A Cowork session and Claude Code started from the same plugin can share one sign-in; if both are open, the one that connected last decides, so the other may get a path with a note or an inline image it did not need.

## Logs and HTTPS problems

**Server disconnects immediately.** Check the log — `~/Library/Logs/Claude/mcp-server-touchdesigner.log` on macOS, `%LOCALAPPDATA%\Claude\logs\` on Windows (older installs wrote to `%APPDATA%\Claude\logs\`; **Settings → Developer** opens the right folder either way). A manual config writes `mcp-server-touchdesigner.log` there. The plugin does not get a file of its own: Desktop records it in `main.log` in the same folder, on lines tagged `[LocalMcpServerManager]` (seen on Windows), and those say only that the connection closed, not why. For the plugin, TDMCP's own event log is the better place to look: `logs/tdmcp_13316.log` in the TDMCP state folder, which the component's **Storage** page opens. On Windows a spawn error means Desktop could not launch `npx` directly, because it is a `.cmd` shim. Wrap it:

```json
"command": "cmd", "args": ["/c", "npx", "-y", "mcp-remote@0.16.6", "http://127.0.0.1:13316/mcp", "--allow-http"]
```

**`UNABLE_TO_VERIFY_LEAF_CERTIFICATE`.** HTTPS only. Node does not trust the mkcert CA; check `NODE_EXTRA_CA_CERTS` points at the right `rootCA.pem`, or switch to plain HTTP.

**`curl` fails on Windows but the client works.** `SEC_E_UNTRUSTED_ROOT` and `CRYPT_E_NO_REVOCATION_CHECK` are a schannel quirk: it demands a revocation endpoint mkcert certs do not publish. Not a real trust failure. Verify with `Invoke-RestMethod` or `curl --ssl-revoke-best-effort`.

## Troubleshooting

### Sign-in never finishes, or the server shows as failed

Desktop starts the plugin's bridge when it launches or when a session opens, and the bridge signs in at once. Sign-in has three steps, and it fails if any one is missed:

1. A browser tab opens on a TDMCP page that says it is waiting for you.
2. TouchDesigner shows the consent panel, with a five-minute countdown and two buttons, **Deny** and **Allow full control**.
3. You press **Allow full control**. The browser tab finishes and the tools load.

**Nothing happens at all, for minutes.** TouchDesigner was not open, or **Active** was off, when Desktop started the plugin. The bridge has nothing to connect to and stops at once, and Desktop tries again later on its own schedule, which can be more than a minute after TouchDesigner is ready. Open the project, turn **Active** on, then start a new session or quit and reopen Desktop. Once it connects, the consent panel appears within a second.

**The tools take a moment after you allow.** About ten seconds pass between pressing **Allow full control** and the tools appearing in the session. That is normal.

**You get one five-minute chance per attempt.** If nobody presses **Allow full control** before the countdown ends, the request expires. Desktop retries a few times, then marks the server as failed and stops trying. From then on a new Cowork session has no TouchDesigner tools, no browser tab opens and no panel appears, which looks as if nothing is happening at all. This is easy to miss when Desktop starts with the computer, or TouchDesigner is behind another window: the tab and the panel came and went while you were elsewhere.

**To get another attempt**, fully quit Claude Desktop and start it again, with TouchDesigner open and **Active** on first. Closing the window is not enough. On Windows use **File → Exit**, or right-click the Claude icon in the system tray and choose Quit. On macOS use **Claude → Quit**. Turning the plugin off and on under **Customize → Plugins** also starts a new attempt. Then watch for the browser tab and switch to TouchDesigner for the panel.

**The panel shows Deny but no Allow full control.** On 1.1.57 the Allow button is sometimes not drawn, so the request cannot be approved and runs out. Let it expire or press **Deny**, then start another attempt as above; the button is usually there the next time. From 1.1.58 both buttons are part of the panel's background image and are redrawn twice a second, so update the component if you see this.

**You closed the browser tab before it finished.** Leave the tab open until it says *Returning you to your application*; the page itself says to keep it open. The tab is what carries the approval back to the client: after you press **Allow full control** it hands the client a one-time code, and a closed tab never delivers it, so sign-in stalls although TouchDesigner shows it as allowed. Start another attempt: in Claude Code run `/mcp` and authenticate again; in Desktop quit and reopen as above. TouchDesigner remembers that you allowed this client, so the second attempt goes through without showing the panel again.

**The browser tab opened but TouchDesigner shows no panel.** The panel is a separate floating window that opens in the middle of the screen your mouse was on when the request arrived, so check your other monitors, and behind other windows. If TouchDesigner was not Active when the tab opened, start another attempt.

**Check what the server saw.** Open the per-port log, `logs/tdmcp_13316.log` in the TDMCP state folder (the **Storage** page shows the path and opens the folder), and read the lines from the time of the attempt:

| The log shows | Meaning |
|---|---|
| `consent_allow` or `consent_reuse`, then `token_issue` | Sign-in worked. If tools are still missing, start a new session |
| `consent_allow` and no `token_issue` after it | Allowed, but the browser tab was closed before it handed the code to the client. Start another attempt; it goes through without the panel |
| `consent_shown`, then `consent_expired` | The panel was shown and nobody approved it in five minutes. Start another attempt |
| `auth_rejected reason=no_token` and no `consent_shown` after it (1.1.58 and later) | The bridge was refused for having no token and did not go on to sign in. Fully quit Desktop and start it again. If it repeats, delete the `~/.mcp-auth` folder (`%USERPROFILE%\.mcp-auth` on Windows) first |
| `auth_rejected reason=no_token` and nothing after (1.1.57 and earlier) | Those versions do not record the panel, so this is either of the two rows above. Start another attempt |
| `consent_deny` | **Deny** was pressed |
| `authorize_reject reason=unknown_client_id` | See [Stuck in a re-authentication loop](troubleshooting.md#stuck-in-a-re-authentication-loop) |
| Nothing at the time of the attempt | The bridge never reached the server. See [The agent has no TouchDesigner tools](troubleshooting.md#the-agent-has-no-touchdesigner-tools) |

`auth_rejected` is written at most once a minute, so one line can stand for several requests.

### Other problems

- **Upload plugin says the archive must contain `.claude-plugin/plugin.json`.** You gave it `tdmcp.mcpb`, or a zip of it. That page takes `derivative_tdmcp.plugin` from the release, which wraps the bundle in a plugin manifest.
- **Nothing under Connectors.** That page lists remote connectors. The plugin's server is on the plugin's own **Connectors** tab under **Customize → Plugins**, marked **Runs in each session**, and shows as **Tdmcp** in a session's connector list.
- **No browser tab, no consent panel, or Desktop shows the server as failed.** See [Sign-in never finishes](#sign-in-never-finishes-or-the-server-shows-as-failed) above.
- **Tools missing after uploading the plugin.** A session starts the bridge when it opens, and TouchDesigner must be Active by then. Make it Active and start a new session; or turn the plugin off and on under **Customize → Plugins**, or fully quit and relaunch Desktop.
- **You uploaded a newer plugin, but a session still reports the old bridge version.** A session keeps the bridge process it started with. Start a new session.
- **Add Custom Connector shows no tools.** It cannot reach a local server at all: connector calls come from Anthropic's cloud. Use `derivative_tdmcp.plugin`, or the [manual config](#manual-config).
- **Authentication error after Clear Registrations.** The bridge caches its client id under `~/.mcp-auth` and keeps presenting it after the server has forgotten it, so every connect is refused. Delete that folder, then turn the plugin off and on; it registers fresh and the consent panel appears once. To cut off access without this, use **Revoke Access** instead: it keeps the registration, so the bridge simply signs in again. See [Revoke Access or Clear Registrations](setup-advanced.md#ending-access-revoke-access-or-clear-registrations).
- **You changed Port.** The plugin cannot be configured and always uses 13316. Put the component back on 13316, or use the manual config.
- **You turned the plugin off, but sessions still have TouchDesigner tools.** Desktop applies the **Disable plugin** toggle to local servers only when it next refreshes its plugin state or restarts, which can be an hour later; until then a running bridge keeps its access and new sessions get one too. To cut access now, press **Revoke Access** on the component's Storage page, or turn **Active** off. Removing the plugin and relaunching Desktop also ends it.
- **You turned on Use HTTPS.** The bundle speaks plain HTTP only, which is safe on loopback, and goes silent when the server switches to TLS. Turn Use HTTPS off; the bundle needs nothing from mkcert. Ask for authentication stays on, as shipped.
- **Desktop says it can't see `view_operator` images, only a file path.** From 1.1.58 a Cowork session gets images inline without any setting, so check the plugin and the component are both 1.1.58 or later. If Claude Code started from the same plugin is open at the same time, it can share the sign-in and the one that connected last decides; start a new Cowork session, or turn on **Inline Images (base64)** on the Tune page to send images inline to everyone. More in [Images](#images).
- **No TouchDesigner tools in a Chat conversation.** Expected: Claude Desktop loads a plugin's local server in Cowork and Code sessions only. Switch the conversation to **Cowork**.
- Desktop's MCP logs are under **Settings → Developer**.
