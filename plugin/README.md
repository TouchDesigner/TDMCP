# TouchDesigner

Lets Claude build and edit [TouchDesigner](https://derivative.ca) projects
with you: create operators, wire them, set parameters, write shaders, and see
what they output. TouchDesigner is a visual programming environment for
real-time graphics, interactive installations and live visuals, made by
[Derivative](https://derivative.ca).

This plugin is half of the connection. The other half is a small component,
`TDMCP.tox`, that runs inside TouchDesigner.

## Get started

1. **Install TouchDesigner** 2025.33230 or later from
   [derivative.ca/download](https://derivative.ca/download). The
   Non-Commercial licence is free with a Derivative account.
2. **Add the component.** Download `TDMCP.tox` from the latest release at
   [github.com/TouchDesigner/TDMCP/releases](https://github.com/TouchDesigner/TDMCP/releases).
   Open TouchDesigner and drag the file from your Downloads folder into the
   network, the large grid area. Click the new `tdmcp` node, press **P** if
   no parameter window is showing, and on its **MCP** page switch **Active**
   on.
3. **Restart Claude Desktop** with TouchDesigner still open. Quit it fully
   (File → Exit on Windows, Claude → Quit on macOS) and open it again. A
   browser tab opens, and TouchDesigner shows a panel asking whether to let
   Claude in. Press **Allow full control** and leave the tab open until it
   finishes.
4. **Use Cowork.** Start a conversation in **Cowork** mode, under the prompt
   box, and ask: *What TouchDesigner project am I connected to?* Then try:
   *Create a noise TOP, blur it, and end the chain in a null.* Plain Chat
   does not load this plugin. Cowork needs a paid Claude plan.

Keep TouchDesigner open with **Active** on while you work.

## What's included

- The connection to the `TDMCP.tox` component, on this computer only.
- 19 skills that teach Claude how TouchDesigner networks are built (TDMCP
  Skills 1.4.3). Nothing to install.

Use the `TDMCP.tox` from the same release as this plugin. To update, upload the
newer plugin over this one and restart Claude Desktop.

## Safety

Claude gets full control of the TouchDesigner project you allow, including
running Python inside it, so treat the connection like a terminal on your
computer. It only connects to TouchDesigner on this machine and sends nothing
anywhere else. Privacy policy: <https://derivative.ca/privacy>.

More help: [Claude Desktop guide](https://github.com/TouchDesigner/TDMCP/blob/main/docs/claude-desktop.md).
