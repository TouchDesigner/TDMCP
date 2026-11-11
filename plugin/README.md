# TDMCP

TouchDesigner for Claude, as one plugin. It carries `TDMCP.mcpb`, the
bridge that connects Claude to the TDMCP component running inside
TouchDesigner on this computer, and the `td-*` skills (TDMCP Skills
1.4.1) that teach Claude how TouchDesigner networks are built.
Claude Desktop starts the bridge for the chat, Cowork and Code sessions that
run here, and all three load the skills.

## Setup

1. Drop `TDMCP.tox` into the root of your TouchDesigner project and turn on
   **Active**.
2. Add **TouchDesigner** from the plugin directory in Claude, or upload
   `Derivative-TDMCP.plugin` from the release with **Upload
   plugin**. Accept the prompt that it runs on your computer.
3. Start a conversation, or a Cowork or Code session, on this computer. On
   the first connection a browser tab opens and TouchDesigner shows a consent
   panel naming the client that is asking. Nothing connects until you decide
   there.

The plugin's version is the TDMCP version. Use it with the `TDMCP.tox` from
the same release. The bridge talks to TouchDesigner on `127.0.0.1` only, on
port 13316; Desktop offers no settings for a plugin-bundled server, so keep
the component's **Port** at its default.

To update, upload the newer plugin over this one and start a new session. A
session keeps the bridge it started with.

## Skills

The skills cover planning, each operator family, GLSL, materials,
components, layout and review. Claude loads one when a task calls for it;
nothing needs installing. If you also installed the skills with the
component's **Install Skills** for Claude Code, both copies load there and
**Skills Status** on the component says so: keep the plugin and uninstall
the component's copy, or the reverse.

## Safety

The server includes `execute_code`, which runs Python inside TouchDesigner, so
treat the connection as you would a shell on your machine. The bridge connects
only to TouchDesigner on `127.0.0.1` and collects nothing. Full statement in
the repository README under **Privacy**; policy at
<https://derivative.ca/privacy>.
