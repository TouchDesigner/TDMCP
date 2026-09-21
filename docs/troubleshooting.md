# Troubleshooting

## The agent has no TouchDesigner tools

1. Check **Active** is on.
2. Test the server directly. A JSON reply means it is up; `401` means it is up
   with authentication on, which is also success:
   ```bash
   curl -X POST http://127.0.0.1:13316/mcp -H "Content-Type: application/json" \
     -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'
   ```
3. Confirm the port in **Add MCP** matches what your client registered.
4. Restart the client. Most only read MCP config at startup.

## Connection refused

- **Use `127.0.0.1`, never `localhost`.** The socket is IPv4-only. `localhost`
  resolves to `::1` first on many systems, and a client that does not retry the
  next address reports a refusal.
- Check nothing else holds the port.
- If you changed **Port**, re-register: **Remove MCP** then **Add MCP**.

## Install Skills reported an error

- **`CERTIFICATE_VERIFY_FAILED`** — TouchDesigner's bundled OpenSSL ships cert
  paths that do not exist on your machine, so HTTPS from inside TD has no trust
  anchors. TDMCP falls back to the `certifi` bundle that ships with TD. If you
  see this, your TD build has no `certifi`; set **Skills Source** to a local
  TDMCPSkills checkout as a workaround and file an issue.
- **`404 ... no such repo, or it is private`** — check **Skills Repo**. Blank
  means `TouchDesigner/TDMCPSkills`.
- **`save the project first`** — project-scoped installs write beside the `.toe`.
  Save the project, or install at **user** scope.
- Anything else — the message carries the underlying reason. Nothing is touched
  on failure; an existing install stays intact.

## Install Skills does nothing at all

Check the project is saved if you are installing at a project scope, and confirm
**Skills Status** changes when you pulse. A component that responds to nothing —
no menu updates, no status changes — is a build older than 1.1.51; update it.

## Skills installed but the agent ignores them

- Read **Installed Locations** and confirm the directory matches the client you
  are actually running.
- Scope matters: a project install only applies to sessions started in that
  directory.
- Restart the agent. Most discover skills at startup.
- For scripted Antigravity, install at **user** scope — see [Skills](skills.md).

## OAuth: 401, or "does not match expected", after a restart or a URL change

Clear the client's stored authentication and reconnect. In Claude Code:
`/mcp` → `touchdesigner` → *Clear authentication* → *Reconnect*, then approve the
TouchDesigner popup. Other clients: remove and re-add the server.

Tokens live in memory, so a TD restart invalidates all of them. The server also
advertises itself exactly as you addressed it, so switching between `localhost`
and `127.0.0.1` invalidates the client's cached identifier.

Restarting TouchDesigner does not help. The stale state is in the client.

## Stuck in a re-authentication loop

The consent page opens but never completes. The client is replaying a `client_id`
this server does not have, and re-authenticating preserves registrations while
clearing only tokens, so it fails identically every time.

Confirm with `grep authorize_reject .tdmcp/oauth_events.log` —
`reason=unknown_client_id` is this failure. Fix by clearing the *registration*:
`claude mcp logout touchdesigner`, or remove and re-add for other clients.

## Claude Desktop problems

See [Claude Desktop](setup-advanced.md#claude-desktop). Note that Desktop's
**Add Custom Connector** cannot reach a local server at all; use the bridge.
