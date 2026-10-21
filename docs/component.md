# Component reference

Everything on the TDMCP component, page by page, plus the tools it serves and
where the skills land. The [README](../README.md) is the short version.

# Parameters

Six pages: **MCP**, **Skills**, **Tune**, **Docs**, **Storage**, **About**.

## MCP — server and agent setup

| Parameter | Notes |
| --- | --- |
| **Active** | Start and stop the server |
| **Port** | Default `13316` |
| **Server Name** | The name clients register the server under. Default `touchdesigner` |
| **Server Address Scope** | `Localhost only (127.0.0.1)` (default) or `All interfaces (LAN/remote)`. See [Security](security.md#security) |
| **Use HTTPS** | Optional TLS. See [HTTPS](setup-advanced.md#https) |
| **Certs Folder** | Optional override for where the cert lives |
| **Ask for authentication** | On by default. OAuth 2.1 with a consent panel inside TouchDesigner. See [Security](security.md#security) |
| **Auto-accept deletion** | Skip the confirmation dialog on `delete_operator`. That is the only tool that raises one |
| **Install For** | Which agent to set up. Default Claude Code |
| **Install Scope** | Rebuilt per agent to the scopes it supports. Default `Local — this folder, private to you` |
| **Add MCP** / **Remove MCP** | Read-only, generated for the current selection |
| **Install Skills** | Install for the current Install For × Install Scope |
| **Skills Status** | Read-only. What is installed for that selection; the tooltip lists resolved directories |

**Running two TouchDesigner instances at once?** Give each component a different **Server Name**. Clients resolve one entry per name, so two servers both called `touchdesigner` means one silently shadows the other and your agent drives the wrong project.

## Skills — content, overrides and cleanup

| Parameter | Notes |
| --- | --- |
| **System prompt DAT** | The DAT served to clients as MCP `instructions`. Defaults to `./SystemPrompt` |
| **Skills Source (blank = latest release)** | Blank downloads the latest published release. A path is used only when it resolves to a TDMCPSkills checkout; a stale path falls back to the release with a note |
| **Skills Repo or Archive URL** / **Skills Version** | Override the repo (`owner/name`, a direct archive URL, or `file://`) and pin a release tag. Blank means `TouchDesigner/TDMCPSkills` at its latest release |
| **Host Override DAT** | A DAT of JSON host records layered over the shipped registry. See [Adding an agent](setup-advanced.md#adding-an-agent) |
| **Installed Locations** | Read-only. Every registry destination currently holding skills. One directory serving two clients is listed once, with both named |
| **Uninstall Scope** | `Current selection only (MCP page)` · `All agents — this project` · `All agents — global` · `All agents — everywhere` |
| **Uninstall Skills** | Remove the skills this component installed, per the manifests it wrote |

> **Every Uninstall Scope except the first ignores Install For** and sweeps every agent at that scope. Read **Installed Locations** before a wide uninstall.

## Tune — rate limiting and visual feedback

**Rate limiting** (`Enable` on, `Slow Threshold` 5.0s, `Cooldown` 1.0s) throttles after a slow tool call. **Inline Images (base64)** (off), **Image Cache Path** (blank) and **Request Log** (off) control image return and logging. With Inline Images off, captures are written as PNGs and the path returned: blank means the TDMCP state folder, `cache/<port>`, cleared when the server starts; a relative path is beside the project; an absolute path is used as given. **Node following** (`Follow Active`, `Follow Time`, `Highlight Nodes`, `Color Time`, `Color Delay`) drives the pan and highlight behaviour.

## Docs — documentation source

**Docs Source** selects where `get_docs` reads from: `Auto (local if installed)` (default), `Web (docs.derivative.ca)`, or `Local only`. **Local Docs Path** points at an offline copy. **Status** is read-only.

## Storage — what TDMCP writes, and where

Everything TDMCP keeps on your machine lives in one folder beside TouchDesigner's own preferences: `%LOCALAPPDATA%\Derivative\TDMCP` on Windows, `~/Library/Application Support/Derivative/TDMCP` on macOS. Nothing is written beside your project. This page shows the exact paths for the running port and lets you open or clear them.

| Parameter | Notes |
| --- | --- |
| **State Folder** / **Open Folder** | The folder itself. **Open Folder** reveals it in Explorer or Finder |
| **Registrations** / **Clear Registrations** | `oauth_clients_<port>.json`: which clients have registered, as public ids and names, no secrets. Beside it, `oauth_tokens_<port>.json` holds their live access tokens. Clearing removes both and ends every session on the port; clients reconnect and consent again. Claude Desktop's bridge also needs its own cache cleared, see [Claude Desktop problems](troubleshooting.md#claude-desktop-problems) |
| **Logs** / **Clear Logs** | `logs/tdmcp_<port>.log`, plus one rotated `.1`: authentication and session events, never tool payloads. Clearing removes every log file |
| **Capture Cache** / **Clear Cache** | `cache/<port>/`: `view_operator` PNGs when Inline Images is off. Emptied automatically when the server starts |
| **Status** | Read-only. What the last action did |

Delete the whole folder to reset TDMCP. It is recreated on the next start.

## About

**Help** opens this repository. **Update** checks for a newer release and tells you whether you need it; if there is one it asks first, then replaces the component in place and keeps your settings. **Version** and **.tox Save Build** are what to quote in a bug report.

# Tools

26 tools, as the agent sees them.

| Tool | What it does |
| --- | --- |
| `project_info` | Project, panes and server details: the first call in any session. |
| `list_operators` | List the operators inside a component. |
| `search_operators` | Find operators by type, family or name across the project. |
| `get_operator_info` | Type, position, size, non-default parameters and connections of one operator. |
| `get_connections` | Wiring of an operator, or the whole signal flow of a component. |
| `get_help` | Parameter names, types and menu values for operator types. |
| `get_docs` | TouchDesigner documentation: the Python API and operator articles. |
| `get_parameters` | Read parameter values from one or more operators. |
| `set_parameters` | Set parameter values on one or more operators. |
| `pulse_parameter` | Pulse a pulse-type parameter. |
| `edit_custom_parameters` | Add, edit, sort, rename or delete custom parameters on a COMP. |
| `create_operator` | Create an operator with its position and parameters. |
| `build_network` | Create and wire several operators in one call. |
| `wiring` | Connect or disconnect operators. |
| `edit_operator` | Rename, copy, move, retype, reposition, flag or color an operator. |
| `reposition_operators` | Move several operators in one call. |
| `delete_operator` | Delete an operator. |
| `annotation` | Create or edit network box annotations. |
| `get_dat_content` | Read a DAT's text or table. |
| `set_dat_content` | Write or edit a DAT, optionally synced to a file on disk. |
| `get_errors` | Cooking errors and warnings for an operator and its children. |
| `inspect_values` | Runtime values of an operator: channels, pixels, points or cells. |
| `view_operator` | Capture an operator's output as an image. |
| `get_cook_performance` | Cook times, to find what is slowing a network down. |
| `execute_code` | Run Python inside TouchDesigner. |
| `save_project` | Save the project file. |

Read-only tools such as `project_info`, `get_help` and `get_parameters` are
marked as such in their MCP annotations, so clients that gate approvals on
annotations can wave them through. Everything else is wrapped in an undo block.

# Skills

Skills teach the agent how TouchDesigner expects things to be done: builder guides per operator family, layout and performance conventions, and workflow skills for planning, reviewing and cleanup. Without them an agent will guess at parameter names and produce networks that look plausible and are wrong.

They live in the separate [TDMCPSkills](https://github.com/TouchDesigner/TDMCPSkills) repo, which is content only. Installation lives here, in the component, and needs no clone: **Skills Source** blank downloads the latest published release.

Where they land depends on the client, because there is no shared directory:

| Client | User scope | Project scope |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex | `~/.codex/skills/` | `.agents/skills/` |
| Antigravity (`agy`) | `~/.gemini/config/skills/` | `.agents/skills/` |
| OpenCode v2 (beta, `opencode2`) | `~/.config/opencode/skills/` | `.agents/skills/` |

Codex, Antigravity and OpenCode v2 share `<project>/.agents/skills/`, so one project install serves all three.

> **Antigravity, scripted:** its workspace skills mount in the interactive TUI but **not** under headless `agy --print`. If you drive `agy` from a script, install at **user** scope.

Cursor and OpenCode v1 have no skills mechanism we have been able to establish, so they are connection-only. OpenCode v2, in beta, reads skills and is the most practical way to try a local model: see [docs/opencode.md](opencode.md).

**The component owns the `td-` namespace.** Each install removes any `td-*` skill *it previously installed* that the current source no longer ships, so renames do not leave orphans. A `td-*` folder it did not install is left alone, as is anything without the prefix.
