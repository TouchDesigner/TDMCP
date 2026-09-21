# The component's parameters

Five pages: **MCP**, **Skills**, **Tune**, **Docs**, **About**.

## MCP — server and agent setup

| Parameter | Notes |
| --- | --- |
| **Active** | Start and stop the server |
| **Port** | Default `13316` |
| **Server Name** | The name clients register the server under. Default `touchdesigner` |
| **Server Address Scope** | `Localhost only (127.0.0.1)` (default) or `All interfaces (LAN/remote)`. See [Security](security.md) |
| **Use HTTPS** | Optional TLS. See [HTTPS](setup-advanced.md#https) |
| **Certs Folder** | Optional override for where the cert lives |
| **Ask for authentication** | OAuth 2.1 with an in-TD consent popup. See [Security](security.md) |
| **Auto-accept deletion** | Skip the confirmation dialog on `delete_operator`. That is the only tool that raises one. **On by default** |
| **Install For** | Which agent to set up. Default Claude Code |
| **Install Scope** | Rebuilt per agent to the scopes it supports. Default `Local — this folder, private to you` |
| **Add MCP** / **Remove MCP** | Read-only, generated for the current selection |
| **Install Skills** | Install for the current Install For × Install Scope |
| **Skills Status** | Read-only. What is installed for that selection; the tooltip lists resolved directories |

!!! tip "Running two TouchDesigner instances at once?"

    Give each component a different **Server Name**. Clients resolve one entry
    per name, so two servers both called `touchdesigner` means one silently
    shadows the other and your agent drives the wrong project.

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

!!! warning "Wide uninstalls ignore Install For"

    Every **Uninstall Scope** except the first sweeps every agent at that scope.
    Read **Installed Locations** before a wide uninstall.

## Tune — rate limiting and visual feedback

**Rate limiting** (`Enable` on, `Slow Threshold` 5.0s, `Cooldown` 1.0s) throttles
after a slow tool call. **Inline Images (base64)** (off), **Image Cache Path**
(`.claude/cache`) and **Request Log** (off) control image return and logging.
**Node following** (`Follow Active`, `Follow Time`, `Highlight Nodes`,
`Color Time`, `Color Delay`) drives the pan and highlight behaviour.

## Docs — documentation source

**Docs Source** selects where `get_docs` reads from: `Auto (local if installed)`
(default), `Web (docs.derivative.ca)`, or `Local only`. **Local Docs Path**
points at an offline copy. **Status** is read-only.

## About

**Help** opens this repository. **Version** and **.tox Save Build** are what to
quote in a bug report.
