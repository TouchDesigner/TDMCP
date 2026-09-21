# Quick start

Three steps: add the component, connect your agent, confirm the round trip.

## 1. Add the component

1. Download `TDMCP.tox` from [Releases](https://github.com/TouchDesigner/TDMCP/releases).
2. Drag it into the root of your TouchDesigner project.
3. Turn on **Active** on the MCP page.

## 2. Connect your agent

Two things happen here: registering the server, so the agent gets the tools, and
installing the skills, so it knows how TouchDesigner expects things to be done.
Both live on the component's **MCP** page.

1. Pick your client in **Install For**. Defaults to Claude Code.
2. Pick an **Install Scope**. The menu rebuilds per client and offers only the
   scopes that client supports.
3. Copy **Add MCP** and run it in your project directory.
4. Pulse **Install Skills**.

For Claude Code that gives you:

```bash
claude mcp add --transport http --scope local touchdesigner http://127.0.0.1:13316/mcp
```

!!! tip "The command is generated, not typed"

    **Add MCP** and **Remove MCP** rebuild from your live **Port**, **Server
    Name**, **Use HTTPS** and **Ask for authentication** settings, so the command
    always matches the server you are actually running.

**Install Skills** downloads the latest published
[TDMCPSkills](https://github.com/TouchDesigner/TDMCPSkills) release, so this step
needs network access. Point **Skills Source** at a local checkout to work
offline.

## 3. Check it worked

```bash
claude mcp list
# touchdesigner: http://127.0.0.1:13316/mcp (HTTP) - ✔ Connected
```

Then ask your agent:

> What TouchDesigner project am I connected to?

A correct answer means the whole chain works: the server is running, the client
found it, and a tool call round-tripped. If it does not, see
[Troubleshooting](troubleshooting.md).

Now try something real:

> Create a noise TOP, blur it, and end the chain in a null.

## Next

- [What you get](what-you-get.md) — the tools and the behaviour around them
- [Skills](skills.md) — what the agent learns before it touches your project
- [Other clients](setup-advanced.md#other-clients) — Codex, Antigravity, Cursor, OpenCode
- [Security](security.md) — what the defaults expose
