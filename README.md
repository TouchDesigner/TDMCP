# TouchDesigner MCP

**An MCP server that runs inside TouchDesigner, so an AI coding agent can collaborate with you in a TouchDesigner environment.**

Drop one component into your project and your agent gets 26 tools for creating operators, wiring them, setting parameters, reading DATs, inspecting runtime values and looking up TD documentation.

Works with Claude Code, Codex, Antigravity (`agy`), Cursor, OpenCode, and any other client that speaks streamable-HTTP MCP.

### 📖 [Documentation → touchdesigner.github.io/TDMCP](https://touchdesigner.github.io/TDMCP/)

> **Beta.** The interface may still change and feedback is wanted. When reporting something, include the **Version** and **.tox Save Build** from the component's About page.
> [Issues](https://github.com/TouchDesigner/TDMCP/issues) · [Discussions](https://github.com/TouchDesigner/TDMCP/discussions)

## Quick start

1. Download `TDMCP.tox` from [Releases](https://github.com/TouchDesigner/TDMCP/releases) and drag it into the root of your project.
2. Turn on **Active** on the MCP page.
3. Pick your client in **Install For**, copy **Add MCP**, and run it in your project directory.
4. Pulse **Install Skills**.

For Claude Code that gives you:

```bash
claude mcp add --transport http --scope local touchdesigner http://127.0.0.1:13316/mcp
```

Confirm it with `claude mcp list`, then ask your agent *"What TouchDesigner project am I connected to?"*

Requires **TouchDesigner 2025.33070 or later**. Full walkthrough in the
[quick start](https://touchdesigner.github.io/TDMCP/quick-start/).

## Documentation

| | |
| --- | --- |
| [Quick start](https://touchdesigner.github.io/TDMCP/quick-start/) | Add the component, connect your agent |
| [What you get](https://touchdesigner.github.io/TDMCP/what-you-get/) | The 26 tools, undo, visual feedback |
| [Parameters](https://touchdesigner.github.io/TDMCP/parameters/) | Every page on the component |
| [Skills](https://touchdesigner.github.io/TDMCP/skills/) | What the agent learns, and where it installs |
| [Security](https://touchdesigner.github.io/TDMCP/security/) | What the defaults expose |
| [Advanced setup](https://touchdesigner.github.io/TDMCP/setup-advanced/) | Other clients, HTTPS, Claude Desktop, OAuth |
| [Troubleshooting](https://touchdesigner.github.io/TDMCP/troubleshooting/) | When it does not connect |

## Skills

Skills teach the agent TouchDesigner conventions before it touches your project. They live in [TDMCPSkills](https://github.com/TouchDesigner/TDMCPSkills) and install from the component's MCP page, or as a Claude Code plugin.

## This repository

Documentation and published builds. Development happens in a separate repository.

## License

Shared Use License — see [LICENSE.md](LICENSE.md). It permits use and modification for your own projects and prohibits redistributing TDMCP itself as a product. Read it before shipping anything built on it commercially.
