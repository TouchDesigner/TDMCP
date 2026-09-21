# Skills

Skills teach the agent how TouchDesigner expects things to be done: builder
guides per operator family, layout and performance conventions, and workflow
skills for planning, reviewing and cleanup. Without them an agent will guess at
parameter names and produce networks that look plausible and are wrong.

They live in the separate
[TDMCPSkills](https://github.com/TouchDesigner/TDMCPSkills) repo, which is
content only. Installation lives here, in the component, and needs no clone:
**Skills Source** blank downloads the latest published release. Claude Code users
can also install them from the plugin marketplace in that repo.

## Where they land

There is no shared skills directory, so the destination depends on the client.

| Client | User scope | Project scope |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex | `~/.codex/skills/` | `.agents/skills/` |
| Antigravity (`agy`) | `~/.gemini/config/skills/` | `.agents/skills/` |

Codex and Antigravity share `<project>/.agents/skills/`, so one project install
serves both.

!!! warning "Antigravity, scripted"

    Its workspace skills mount in the interactive TUI but **not** under headless
    `agy --print`. If you drive `agy` from a script, install at **user** scope.

Cursor and OpenCode have no skills mechanism we have been able to establish, so
they are connection-only.

## The `td-` namespace

**The component owns the `td-` namespace.** Each install removes any `td-*` skill
*it previously installed* that the current source no longer ships, so renames do
not leave orphans. A `td-*` folder it did not install is left alone, as is
anything without the prefix.

Author your own skills under your own prefix — anything named `td-*` may be
replaced or removed without warning.

## Project scope needs a saved project

Every scope but **user** resolves against the project folder. A project that has
never been saved still reports one, so the skills would land somewhere the agent
never looks. Installing at a project scope without a saved `.toe` is refused with
a message rather than failing silently.
