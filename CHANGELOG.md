# Changelog

All notable changes to TDMCP are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); the
project uses [Semantic Versioning](https://semver.org/). Each release is tagged
with the bare version baked into that build, so a `.tox` can always be traced to
its release: read **Version** on the component's About page and find the
matching tag.

This changelog begins at the first public release. Earlier versions were
internal builds and were never published.

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
