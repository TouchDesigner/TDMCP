# Changelog

All notable changes to TDMCP are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); the
project uses [Semantic Versioning](https://semver.org/). Each release is tagged
with the bare version baked into that build, so a `.tox` can always be traced to
its release: read **Version** on the component's About page and find the
matching tag.

This changelog begins at the first public release. Earlier versions were
internal builds and were never published.

## [1.1.50] — first public beta

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
