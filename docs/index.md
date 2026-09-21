---
hide:
  - navigation
---

# TouchDesigner MCP

**An MCP server that runs inside TouchDesigner, so an AI coding agent can collaborate with you in a TouchDesigner environment.**

Drop one component into your project and your agent gets 26 tools for creating
operators, wiring them, setting parameters, reading DATs, inspecting runtime
values and looking up TD documentation.

Works with Claude Code, Codex, Antigravity (`agy`), Cursor, OpenCode, and any
other client that speaks streamable-HTTP MCP.

[Get started](quick-start.md){ .md-button .md-button--primary }
[Download the .tox](https://github.com/TouchDesigner/TDMCP/releases){ .md-button }

!!! info "Beta"

    The interface may still change and feedback is wanted. When reporting
    something, include the **Version** and **.tox Save Build** from the
    component's About page — they identify the exact build you are running.

    [Issues](https://github.com/TouchDesigner/TDMCP/issues) ·
    [Discussions](https://github.com/TouchDesigner/TDMCP/discussions)

## One component, no separate process

TDMCP is a single `.tox`. It runs the server inside TouchDesigner on
TouchDesigner's own Python — there is no bridge, no sidecar process and nothing
else to install.

<div class="grid cards" markdown>

-   :material-flash-outline:{ .lg .middle } **Quick start**

    ---

    Add the component, register your agent, confirm the round trip.

    [:octicons-arrow-right-24: Three steps](quick-start.md)

-   :material-toolbox-outline:{ .lg .middle } **What you get**

    ---

    26 tools, undo on edits, and a network editor that follows the agent.

    [:octicons-arrow-right-24: The tools](what-you-get.md)

-   :material-school-outline:{ .lg .middle } **Skills**

    ---

    TouchDesigner conventions the agent loads before it touches your project.

    [:octicons-arrow-right-24: How they install](skills.md)

-   :material-shield-lock-outline:{ .lg .middle } **Security**

    ---

    The server exposes `execute_code`. Know what the defaults do.

    [:octicons-arrow-right-24: What is exposed](security.md)

</div>

## Requirements

| | |
| --- | --- |
| **TouchDesigner** | Tested against build 2025.33070 |
| **An MCP client** | Claude Code needs nothing else and is the simplest starting point |
| **Node.js 18+** | Only for Claude Desktop, which needs a bridge. No other client requires it |
| **mkcert** | Only if you want HTTPS, which is optional |

## Where it is useful

Agents are good at Python and GLSL, at reading structured state, and at checking
their own results, so that is what TDMCP is built around. An agent can sync a DAT
to a file on disk, edit that file with its normal tools while TouchDesigner
reloads it, then check errors and sample the running output to confirm the change
worked. Code on disk plus verification through the live process is where the
value is today. Agents are also good at analysing an existing project and
answering questions about it.

## Where it is less tested

Building networks from scratch has not been the focus. TouchDesigner is a visual
language, and much of what makes a network good is how it reads to a person at a
glance, which an agent working through tool calls cannot see. Agents can still
produce useful networks, and results depend a lot on the model, but expect more
variation here than with code and analysis. The [skills](skills.md) reflect
this: they cover extensions, shaders, callbacks, debugging, review and localised
edits, and go lighter on autonomous network building.
