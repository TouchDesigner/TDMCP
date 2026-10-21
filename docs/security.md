# Security and privacy

# Security

The server exposes `execute_code`. Treat reaching it as equivalent to a shell on your machine.

**Server Address Scope** controls who can:

- **Localhost only** (default) — any address outside loopback (`127.0.0.0/8`, `::1`) is refused `403 Forbidden` before routing, authentication or any tool runs. On TD 2025.33060+ the socket is also bound to `127.0.0.1`, so a port scan from another host sees nothing.
- **All interfaces** — non-loopback clients are answered **only when Ask for authentication and Use HTTPS are both on**. Either one off and off-host requests are refused `403`. Unauthenticated LAN access would hand out `execute_code`; un-TLS'd LAN access would put bearer tokens on the wire in cleartext.

**Origin checking** is always on. A request carrying an `Origin` header that is neither loopback nor this server's own origin is refused `403`. This is what stops a web page you have open from reaching the server on your behalf.

**Auto-accept deletion** bypasses the one confirmation dialog TDMCP raises. Leave it off unless you are running tests.

**Authentication** is on by default: OAuth 2.1 with PKCE and a consent panel inside TouchDesigner. Nothing on the machine gets control without a person clicking Allow in that panel. The panel is a small floating window, not a modal dialog: the server keeps answering other clients while it is open, a request nobody answers expires after five minutes, and when the same application asks twice at once, as Claude Desktop does on connect, one answer covers both. It appears once per client per TouchDesigner session: a client you have allowed gets its later authorizations without another dialog until TD restarts or you clear registrations. Access tokens last 24 hours and survive a TD restart; they live per port in the TDMCP state folder (`oauth_tokens_<port>.json`) beside the client registrations (`oauth_clients_<port>.json`), so every project served on that port shares them and a Save As changes nothing. Details in [OAuth](setup-advanced.md#oauth).

# Privacy

TDMCP collects nothing and sends nothing home. Everything below stays on your machine and is yours to delete.

- **No telemetry, no analytics, no account.** The component and the Claude Desktop bundle do not log or transmit tool calls, parameters, code, or any conversation content.
- **Outbound connections, only when you trigger them:** GitHub (`github.com`, `api.github.com`) when you pulse **Install Skills** with no local source, or **Update** on the About page; `derivative.ca` when `get_docs` falls back to the live wiki. Nothing connects at startup.
- **Local state:** OAuth client registrations, a per-port event log and the capture cache live in one folder beside TouchDesigner's own preferences: `%LOCALAPPDATA%\Derivative\TDMCP` on Windows, `~/Library/Application Support/Derivative/TDMCP` on macOS. Nothing is written beside the project. Access tokens are kept in the same folder for their 24-hour lifetime, in a file only your user account can read. The component's **Storage** page shows these paths, opens the folder, and clears each one; deleting the folder resets TDMCP. The Desktop bundle's `mcp-remote` keeps its own OAuth state in `~/.mcp-auth`.
- **The bridge talks to TouchDesigner on `127.0.0.1` only** and cannot be pointed elsewhere.

Derivative's privacy policy covers the website and services: <https://derivative.ca/privacy>. Privacy questions: info@derivative.ca.
