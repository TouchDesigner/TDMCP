# Changelog

All notable changes to TDMCP are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); the
project uses [Semantic Versioning](https://semver.org/). Each release is tagged
with the bare version baked into that build, so a `.tox` can always be traced to
its release: read **Version** on the component's About page and find the
matching tag.

This changelog begins at the first public release. Earlier versions were
internal builds and were never published.

## [1.1.58] — 2026-11-14

### Claude Desktop needs Cowork

TouchDesigner's tools load in Claude Desktop's **Cowork** sessions and its
**Code** tab, not in plain **Chat**: Desktop does not load a plugin's local
server in Chat. Cowork needs a paid Claude plan. The docs said "chat, Cowork
and Code"; they now say Cowork throughout, and the new
[Claude Desktop](docs/claude-desktop.md) page gathers everything about the
plugin, the Code tab, images and Desktop troubleshooting.

### Added
- **The event log records what the textport says.** `logs/tdmcp_<port>.log`
  in the state folder is now the record to send with a bug report; the
  textport is gone when TouchDesigner closes. New events: `server_start`
  (component version, TouchDesigner build, scheme, port, address scope,
  authentication on or off), `server_stop`, `server_blocked`, `https_cert`,
  `request_refused` (non-local client, oversized body, cross-origin),
  `rate_limited`, `consent_cap`, `consent_panel_error`, `self_path_refused`,
  `request_error`, `refresh_error`, `registry_error` and `bridge_mismatch`.
  Events a caller without a token can trigger are written at most once a
  minute.
- **`consent_shown` and `consent_expired`** in the event log
  (#15). A sign-in nobody approved used to leave only
  `auth_rejected` lines, which is also what a bridge that never began
  sign-in leaves.
- **`get_cook_performance` takes `max_age_frames`** (default 10) and reports
  `framesSinceCook` on every entry; see Fixed.
- **Troubleshooting: Claude Desktop sign-in that never finishes.** One
  five-minute chance per attempt, how to get another (quit Desktop from the
  tray on Windows), and what the log shows in each case.

### Changed
- **Renamed: the plugin is `derivative_tdmcp.plugin`**, and the bridge inside
  it `tdmcp.mcpb`, in lower case with underscores. Upload the new file over
  the old plugin as usual.
- **`TDMCP.tox` and `TDMCP.json` keep their names for this release** and
  become `tdmcp.tox` and `tdmcp.json` in 1.1.59. Components up to 1.1.57
  download them by exact name, and a GitHub release cannot hold both
  spellings, so the rename waits one release: **Update** in 1.1.58 reads
  either name. If a script or link downloads them by name, be ready to
  change it.
- **Docs by audience.** The README is for coding agents (Claude Code, Codex,
  Cursor, Antigravity, OpenCode). Claude Desktop has its own page. The release
  notes are a short Claude Desktop setup. The plugin's README, which Desktop
  shows when someone installs the plugin, now starts from what TouchDesigner
  is and where to get it and `TDMCP.tox`.
- **mcp-remote 0.16.6** in the Claude Desktop bridge and in the generated
  `claude mcp add` command, up from 0.15.1. It brings three security fixes
  (zod outside GHSA-mrhx-c7xc-m3w5, a patched fast-uri, and CVE-2026-108716:
  https required for the device grant, and credentials no longer follow a
  redirect to another origin), opens the browser for a fresh registration
  after a stale one is cleared, and redeems a refresh token once after a
  401. Its `body-parser` is updated to 1.20.9 (GHSA-27r3-jxmh-p828); the
  bridge's dependencies audit clean.
- **The textport stays quiet.** A working server prints one line there,
  when it starts listening, with its URL; warnings, setup problems and errors
  still print. Routine activity (sign-ins, starts and stops, cleared
  captures, refused requests) goes to the console instead: the console
  window on Windows, or the terminal TouchDesigner was started from. Lines a
  caller can repeat, such as refused cross-origin or non-local requests and
  rate-limit refusals, print at most once a minute for each kind, and a
  request error prints its traceback once a minute and a one-line summary
  otherwise. The event log still records everything.
- **`view_operator` returns images by caller.** A Claude Desktop Cowork
  session, which cannot open files, now gets captures inline without
  any setting; before, it got only a file path and could not see what it
  built. Claude Code, by any route, still gets the path and opens the file
  when it needs to look, so its token use is unchanged. Any other client gets
  the path with a one-line note naming **Inline Images (base64)**, which still
  sends images inline to everyone when on. The tool description no longer
  promises an inline image.
- **Operators inside the component are snake_case**.
  The component is `tdmcp` and its tools folder is `tools`; inside it are
  `view`, `menu_extraction`, `mcp_server_ext`, `oauth_provider`,
  `follow_system`, `skills_installer`, `hosts_registry`, `updater`, `hosts`,
  `menu_values` and `system_prompt`. **System prompt DAT** defaults to
  `./system_prompt`. Class names are unchanged, so `ext.MCPServerExt` still
  resolves, and so do the parent shortcut `parent.TDMCP`, `TDMCP.tox`,
  `TDMCP.json` and the state folder. **Update** keeps the name of the
  component it replaces, so an existing `TDMCP` stays `TDMCP`. Source files
  take the operator names: `TDMCP/mcp_server_ext.py` and so on.
- **One skills source for Claude** (#5). For a Claude
  client whose account plugin already provides the `td-*` skills, **Install
  Skills** installs nothing, since a second copy lists every skill twice,
  and **Skills Status** reads `Provided by the account plugin` with the
  skills version. Setting **Skills Source** or **Skills Version** is a
  deliberate choice and still installs.
- **The plugin ships TDMCP Skills 1.4.3** and says so in a manifest inside
  its `skills/` folder, the same one **Install Skills** writes. Skills
  Status shows `v1.4.3 (account plugin)`; it used to show the plugin's
  `plugin.json` version, the TDMCP version, beside a skills version. A
  plugin without the manifest is listed as `TDMCP 1.1.57 plugin`.
- **`bridgeVersion` is the caller's bridge.** `project_info` reports the
  version of the bridge that sent the request, not the last one any bridge
  sent, and a client with no bridge gets no `bridgeVersion`. After a plugin
  upgrade two bridge versions talk to one server; a session on the current
  bridge was told it was on the old one. The mismatch warning prints, and is
  logged, once per mismatched version.
- **Serial ports and audio output channels are no longer in the menu
  catalog.** They are machine state, like device and font menus.

### Fixed
- **Install Scope starts on Local and does not fall back to User.** The
  component now ships with the documented defaults, Claude Code and Local. When a change of Install For leaves the scope unsupported,
  it lands on Local, then Project, and User only for an agent with nothing
  narrower. A user-scope registration answers in every folder, so a second
  project's own `touchdesigner` could be shadowed or confused with it.
- **The consent panel opens in the middle of the screen the mouse is on.**
  It opened under the mouse, so with the mouse near a screen edge its
  buttons were out of reach and it had to be dragged; the next request then
  put it back under the mouse.
- **The consent panel's countdown could start part-way down.** The bar's
  full width was learned from the bar itself, so a component reloaded while
  a panel was counting down took the shrunk width as full, and every later
  request started there. The full width now comes from the panel's layout.
- **`get_help` reported types and components that do not exist**
  (#13). It trusted TouchDesigner's bundled
  `TDParameterHelp.json`, which lags the build: on 2025.33230 it typed 711
  of 15,661 parameters differently from the operator, listed 728 that do not
  exist and omitted 766 that do. `gridPOP size` was given a `sizew`,
  `circlePOP rad` a `radz`, and `levelTOP clampinput`, a three-entry menu,
  was a Toggle with `components`. The catalog extracted from the running
  build now records each operator's real tuplet components and the style and
  label of every parameter, and `get_help` takes names and types from it
  whenever the catalog matches the running build, keeping the JSON for
  summaries. On another build the JSON stays the list.
- **`get_cook_performance` ranked operators that were not cooking**
  (#14). `cookTime` is the duration of the last cook,
  however long ago, so an operator that cooked once at creation topped the
  list for good. One that has not cooked within `max_age_frames` is now
  marked `stale`, carries no `budgetPct`, and sorts after the operators that
  are cooking; `staleCount` says how many. `budgetPct` and the default sort
  count CPU plus GPU time, where they counted CPU alone.
- **Consent panel: Allow full control could be missing**. On some Windows machines neither button
  plate was drawn, and Allow's dark label vanished on the dark panel, so the
  request could not be approved. The panel's background image now carries
  both plates, and the plates are redrawn when the panel opens and twice a
  second after. Not reproduced, so this is defensive.
- **The duplicate-skills warning fired on a replaced plugin.** Uploading a
  plugin again syncs it into `touchdesigner~g2` beside the old
  `touchdesigner` folder, which stays on disk. Skills Status counted both
  and warned of `td-* skills in 2 places` to anyone who had updated the
  plugin once. Only the newest generation is counted.
- **Connected Clients cut the Desktop bridge's name mid-bracket**, as
  `...tdmcp (via mcp-remote  via TDMCP.mcpb 1.1.57`. It now reads
  `...tdmcp via tdmcp.mcpb 1.1.57`.
- **Plugin text files ship with LF line endings** whatever the skills
  archive carries. The 1.4.1 archive was built on Windows and turned every
  skill file in the public repo's `plugin/` folder into a whitespace-only
  diff.

## [1.1.57] — 2026-11-10

### Added
- **One Claude plugin, `Derivative-TDMCP.plugin`,** replaces both
  `TDMCP.mcpb` and the separate skills zip. It carries the bridge that
  connects Claude Desktop to TouchDesigner and the 19 `td-*` skills (TDMCP
  Skills 1.4.1), and Desktop runs the bridge for chat, Cowork and Code
  sessions on your computer, all of which load the skills. Upload it under
  **Customize → Plugins → Add → Upload plugin**; Desktop no longer has an
  Extensions page. To update, upload the newer plugin over the old one and
  start a new session, since a session keeps the bridge it started with.
- **Install For: Claude Desktop (Code tab)** (#8, #4). Generates the
  `mcp-remote` registration the Code tab needs, with `cmd /c npx` on Windows,
  and installs skills to Claude Code's folders, since the Code tab is Claude
  Code.
- **Version reporting across component and bridge** (#9). `project_info`
  reports `componentVersion` beside `serverVersion`, and `bridgeVersion` once
  a Claude Desktop bridge connects; a mismatch adds a `versionWarnings` entry
  and prints a textport warning.
- **Revoke Access** on the Storage page. Ends every session on the port
  without forgetting the clients: each one signs in again on its next
  request and the consent panel asks again. Unlike **Clear Registrations**,
  the Claude Desktop bridge recovers on its own, so this is the everyday way
  to cut off access.
- **Connected Clients** on the Storage page (#10). The clients that sent a
  request in the last 10 minutes, most recent first, with the name and
  version each reports.
- **Duplicate skill copies are reported** (#6). Skills Status warns when the
  selected agent loads `td-*` skills from more than one place for this
  project, including a Claude account plugin, and the tooltip lists each copy
  with its path and version. Nothing the component did not install is
  removed.
- **Project Root** on the Skills page (#7). Blank means the git root above
  the `.toe`.
- **Use HTTPS says what it breaks.** Flipping it prints the new server URL,
  says clients registered with the old scheme must be re-registered, and
  notes that the Desktop plugin's bridge speaks HTTP only. While it is on,
  **Add MCP**'s tooltip gives `NODE_EXTRA_CA_CERTS` with the real root CA
  path, and the Code tab command sets it for its bridge.

### Security
- **Tools cannot change the server's own component.** Every mutating tool
  refuses a target that is the component or anything inside it, with a
  `self_path` error, however the path is written. Before, `set_parameters`
  could turn **Ask for authentication** off. Read-only tools still see the
  component. `execute_code` is not fenced and is documented as such.

### Changed
- **`project_info` names the running build and the save build apart** (#12).
  `tdBuild`/`tdVersion` are listed first; `saveBuild`/`saveVersion` are now
  `projectSaveBuild`/`projectSaveVersion`, and `buildMismatch: true` appears
  when the two differ.
- **Project-scope skills install at the project root** (#7): the nearest
  folder above the `.toe` that holds `.git`, else the `.toe` folder, or
  **Project Root** when set. An older install beside the `.toe` shows in
  Installed Locations as `beside .toe, not loaded`, and the project-wide
  Uninstall removes it.
- **The BETA badge on the logo has no box.**

### Fixed
- **Changing Port loads that port's registrations.** Before, every client
  registered on the new port was refused as `unknown_client_id` until
  TouchDesigner restarted.
- **Clear Registrations stays cleared across a restart** on a project an
  older build had used.
- **The stale-client page names the Desktop fix** after Clear Registrations:
  delete `~/.mcp-auth`, then turn the plugin off and on.
- **The bridge bundle contains `LICENSE.md`.**

### Documentation
- **HTTPS first run.** The HTTPS section opens by saying it is not needed on
  localhost; `Connecting over HTTPS` explains why Node-based clients refuse a
  certificate the browser accepts and gives `NODE_EXTRA_CA_CERTS` for each
  platform; troubleshooting has `UNKNOWN_CERTIFICATE_VERIFICATION_ERROR` and
  `Recovering from Use HTTPS`.
- **The delete confirmation stays modal, and the docs say so.** Turn on
  **Auto-accept deletion** for unattended installs.
- **Claude Desktop Code tab** (#4): the registration that signs in there.
- **Disabling the plugin in Claude Desktop does not stop a running bridge**
  until Desktop refreshes its plugin state or restarts. Use **Revoke Access**
  to cut access now.
- **`mcp-remote` pinned to `0.15.1`** in the manual config examples.

## [1.1.56] — 2026-10-20

### Added
- **Claude Desktop.** The release carries `TDMCP.mcpb`, a Desktop extension
  that connects Claude Desktop to TouchDesigner. Drag it onto **Settings →
  Extensions** and TouchDesigner's tools appear in Desktop; no Node, config
  file or terminal on your side, and the connection is fixed to loopback.
  The skills come as a plugin upload, `TDMCP-Skills-1.4.1.zip`, from
  **Settings → Plugins → Add → Upload a plugin**.
- **Storage page on the component.** Shows exactly what TDMCP writes on this
  machine and where: the state folder, this port's registrations file, the
  logs folder and this port's capture cache. **Open Folder** reveals the
  folder; **Clear Registrations**, **Clear Logs** and **Clear Cache** empty
  each one. Clearing registrations ends every session: clients reconnect and
  consent again.

### Fixed
- **`save_project` under Increment Filename when Saving (#67).** An explicit
  path is now written exactly where asked, a default save honours the
  preference, and the response names the file TouchDesigner actually wrote. A
  path that would replace a different existing file is refused unless
  `overwrite` is true.
- **Generated HTTPS certs cover `::1` (#62),** matching what a bare
  `mkcert localhost` produces.
- **`get_cook_performance` on a single operator (#64)** no longer raises.
- **Skills follow `CLAUDE_CONFIG_DIR` and `CODEX_HOME` (#66)** when those
  variables are set in TouchDesigner's environment.

### Changed
- **Ask for authentication ships on.** Every client goes through a consent
  panel inside TouchDesigner before it gets control. Turn it off on the
  component for a silent local setup.
- **The consent dialog is a floating panel, not a modal dialog (#59).** The
  server keeps answering while it is open, an unanswered request expires
  after five minutes, and two requests from the same application (Claude
  Desktop starts its extension twice) are answered with one click.
- **Access tokens survive a restart, and consent is asked once per client per
  session (#60).** The 24-hour token lifetime is real now: a TouchDesigner
  restart no longer sends every client back through the consent panel.
- **Runtime state moved out of the project folder (#68).** Registrations, the
  event log and the capture cache live in one folder beside TouchDesigner's
  own preferences: `%LOCALAPPDATA%\Derivative\TDMCP` on Windows,
  `~/Library/Application Support/Derivative/TDMCP` on macOS. A `.tdmcp/`
  folder left by 1.1.55 is imported once and never touched again. **Image
  Cache Path** now ships blank, meaning that per-port cache.
- **Deferred work keeps running while the timeline is paused.** The consent
  countdown, image captures and follow animations count frames on the UI
  clock, so pausing the project no longer freezes them.
- **The browser page shown during authentication matches the consent panel.**
- **The Desktop extension describes itself:** its page in Desktop lists every
  tool, the setup steps, and the TDMCP version it belongs with.
- **Requires TouchDesigner 2025.33230 or later.**

## [1.1.55] — 2026-09-27

### Changed
- **The component's logo is a small embedded image.** 1.1.54 drew it with a
  Web Render TOP, which started a browser process in every project the
  component was dropped into. It is now a PNG inside the component (about
  13 KB) shown by a Movie File In TOP, so nothing runs in the background.

## [1.1.54] — 2026-09-24

### Added
- **The component has a logo.** Its node in the network shows the TDMCP lockup
  with a **BETA** badge and the component's version, so you can see which build
  a project carries without opening the About page.

## [1.1.53] — 2026-09-24

### Fixed
- **Docs Status showed the wrong source.** It only changed when `get_docs` ran,
  so 1.1.52 said `🌐 web` even on a Mac whose install has the local docs, until
  the first lookup. It is now worked out when the component starts and when
  **Docs Source** or **Docs Path** changes. Docs were read locally all along;
  only the label was wrong.
- Local docs are found under the macOS bundle's `Learn/OfflineHelp` spelling
  too, so a case-sensitive volume no longer falls back to the web.

## [1.1.52] — 2026-09-24

### Added
- **Update** on the About page checks for a newer release and tells you whether
  you need it. If there is one, it asks before doing anything, then replaces the
  component in place and keeps your settings. The download is checked against
  the release's `TDMCP.json` manifest, and a `.tox` saved by a newer
  TouchDesigner than yours is refused. Nothing checks at startup. 1.1.51 has no
  Update button, so update from it by hand once.
- **OpenCode v2 (beta)** is its own entry in **Install For**, beside OpenCode
  v1. v2 takes skills, registers per project or with `--global`, and nests its
  config under `mcp.servers`. See [docs/opencode.md](docs/opencode.md), which
  also covers running a local model with Ollama and why `"codemode": false`
  matters for small models.

### Changed
- **Install For** lists every host the component knows, including any added
  through **Host Override DAT**.
- OpenCode v1 offers only **User** scope, because its CLI writes only the global
  config. Project scope used to show that same global command beside a hint
  naming `./opencode.json`, which the command never wrote.

## [1.1.51] — 2026-09-21 — first public beta

### Fixed
- **The component's controls did nothing in a dropped-in copy.** Every pulse and
  value change routes through a callbacks DAT, and the export left that DAT
  switched off in the file it wrote — so **Install Skills**, **Help**, the
  **Active** toggle and every menu were inert, with no error and no status
  change. **1.1.50 is affected; use 1.1.51.**
- **Installing skills into an unsaved project now says so.** Every scope but
  `user` resolves against the project folder, and a project that has never been
  saved still reports one — so the skills went to a directory the agent never
  looks in, and the button appeared to do nothing.
- **`view_operator` captures at the source's aspect ratio.** The presets were
  fixed 16:9, so a square TOP came back letterboxed — black bars, and image
  pixels no longer mapping to panel coordinates for a follow-up click — while a
  panel COMP came back stretched. The preset is now a pixel budget and the
  source decides the shape, so the same capture costs the same. A source smaller
  than its budget is captured at its own size rather than enlarged. `resolution`
  in the response reports what was actually captured.

### Changed
- **Auto-accept deletion ships on.** `delete_operator` no longer raises a
  confirmation dialog by default. The dialog interrupted every build, and an
  agent that can reach `delete_operator` already has `execute_code`. Turn
  **Auto-accept deletion** off on the MCP page to be asked.
- The component opens on the **MCP** page, where the server controls and the
  generated client commands are.

## [1.1.50] — 2026-09-20 — withdrawn

### Added
- **Any MCP client, not just Claude.** The component models Claude Code, Codex,
  Antigravity (`agy`), Cursor and OpenCode, and generates the exact registration
  command or config entry for whichever you pick. Client support is data rather
  than code, so a new client is a JSON record: see
  [Adding an agent](docs/setup-advanced.md#adding-an-agent).
- **Skills install with no clone.** Pulse **Install Skills** and the component
  downloads the latest published
  [TDMCPSkills](https://github.com/TouchDesigner/TDMCPSkills) release. Point
  **Skills Source** at a checkout when you want to work against local edits.
  Installs are manifest-tracked, so an uninstall removes exactly what was
  installed and leaves skills it did not install alone.
- **Server Name**, so two TouchDesigner instances can be registered at once.
  Clients resolve one entry per name; without distinct names one silently
  shadows the other and the agent drives the wrong project.
- **Offline documentation.** The Docs page selects where `get_docs` reads from:
  a local TouchDesigner help install, the web, or auto.

### Security
- **Consent cannot be granted by the caller.** A `POST /authorize` route passed
  its raw request body to the consent handler, which reads the approve/deny
  decision from that body — so a caller could register a client, approve itself,
  and obtain a token for `execute_code` without the TouchDesigner popup ever
  appearing. Enabling authentication was what mounted the endpoint. The route is
  removed; consent comes from the popup and nowhere else.
- **A web page can no longer reach the server.** The request gate allowed a
  missing `Content-Type`, which a cross-origin `no-cors` request can produce,
  and pages served from a loopback address were treated as trusted. Together
  those let a page open in your browser call tools. The header must now be
  present and JSON, which forces a preflight this server never answers.
- **Skill downloads must use HTTPS, and installs are recorded.** Skills become
  instruction files your agent reads in every later session, so `Skills Repo`
  no longer accepts a plain `http://` URL, the manifest records the exact URL
  and SHA-256 of what was installed, and the status says so when skills come
  from somewhere other than the default repository. Downloads are protected by
  TLS but are not signed — the record makes an install auditable rather than
  proving it genuine.
- **The consent dialog shows the real destination.** It was built from the URL's
  authority, which can carry a userinfo prefix, so a redirect registered as
  `https://claude.ai@evil.example/cb` displayed as `claude.ai@evil.example` while
  the browser would navigate to `evil.example`. Such URIs are now refused at
  registration, the dialog reads the host itself, and the redirect is length
  capped so it cannot push the prompt out of view. Bidi control characters are
  also stripped from the displayed client name.
- **Oversized requests are refused.** TouchDesigner's web server has no body
  limit of its own and JSON parsing runs on the main thread, so a large request
  could stall the application — before authentication. Bodies over 8 MB now get
  `413`, and the skills download is bounded against an oversized archive or a
  zip bomb.
- **Serving beyond localhost now fails closed.** `All interfaces` is refused
  with `403` unless authentication *and* HTTPS are both enabled. Unauthenticated
  LAN access would expose `execute_code`; un-TLS'd LAN access would put bearer
  tokens on the wire in cleartext.
- **Origin checking**, always on. A request carrying an `Origin` header that is
  neither loopback nor the server's own is refused, which is what stops a web
  page you have open from reaching the server on your behalf.
- **The consent dialog cannot be authored by the caller.** Client registration
  is unauthenticated, and the client-supplied name reaches the approval prompt
  that gates `execute_code`. Names are collapsed to one line and length-capped,
  and the dialog separates caller-supplied text from facts the caller cannot
  choose.
- On TouchDesigner 2025.33060 and newer, `Localhost only` binds the listening
  socket to `127.0.0.1`, so the port is not visible to a scan from another host.

### Fixed
- Skills downloads failed on macOS with `CERTIFICATE_VERIFY_FAILED`, because
  TouchDesigner's bundled OpenSSL reports certificate paths that exist only on
  the machine it was built on. The installer now falls back to the `certifi`
  bundle that ships with TouchDesigner.
- The component no longer keeps a link to the `.tox` it was dropped in from.
  That path breaks as soon as a project is saved elsewhere or the file moves to
  another machine, and a stale link could restore an old build over your
  component.
- Installing skills targets the component whose button was pulsed. Previously a
  renamed component, or one inside a container, handed the install to whichever
  component answered at the default path.
- Installing for several agents in a row lands each in its own destination. The
  download completes a frame later, and the destination was being re-read at
  that point rather than captured when the button was pressed.

### Documentation
- README rewritten for beta, with advanced setup split into
  [docs/setup-advanced.md](docs/setup-advanced.md): non-default clients, HTTPS,
  Claude Desktop, running multiple instances, OAuth internals.
