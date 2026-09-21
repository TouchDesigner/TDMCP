# Security

The server exposes `execute_code`. **Treat reaching it as equivalent to a shell
on your machine.**

## Who can reach it

**Server Address Scope** controls this.

=== "Localhost only (default)"

    Any address outside loopback (`127.0.0.0/8`, `::1`) is refused
    `403 Forbidden` before routing, authentication or any tool runs. On TD
    2025.33060+ the socket is also bound to `127.0.0.1`, so a port scan from
    another host sees nothing.

=== "All interfaces"

    Non-loopback clients are answered **only when Ask for authentication and Use
    HTTPS are both on**. Either one off and off-host requests are refused `403`.

    Unauthenticated LAN access would hand out `execute_code`; un-TLS'd LAN access
    would put bearer tokens on the wire in cleartext.

**Origin checking** is always on. A request carrying an `Origin` header that is
neither loopback nor this server's own origin is refused `403`. This, with the
content-type check, is what stops a web page you have open from reaching the
server on your behalf.

## What the tools can do

**File writes.** DAT file writes are confined to the project folder.
`execute_code` is arbitrary Python: it is not path-restricted, and on the default
localhost configuration it runs without a prompt.

**Deletion.** **Auto-accept deletion** is on by default, so `delete_operator`
does not raise a confirmation dialog. That is the only tool that raises one at
all. Turn it off on the MCP page if you want to be asked.

**Tool annotations.** Each tool carries annotations describing its effects —
read-only, mutating, destructive and others — so clients can apply their own
approval rules.

**Undo.** Changes made through the editing tools are wrapped in undo blocks, with
annotations as the current exception.

## Authentication

OAuth 2.1 with PKCE and an in-TD consent popup. Access tokens live in memory and
last 24 hours or until TouchDesigner restarts, whichever comes first. Client
registrations persist in `.tdmcp/` beside the project. Details in
[OAuth](setup-advanced.md#oauth).
