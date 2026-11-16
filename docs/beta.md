# Beta status

TDMCP is in public beta. This page says plainly what Derivative has tested,
what it has not, and what we would like to hear about. Setups differ more than
we can cover, so reports from your machine are how the untested cases get
settled.

## What we test

Each release is tested on Windows 11 and macOS with TouchDesigner 2025.33230:

- **Claude Code**, connected with **Add MCP**.
- **Claude Desktop** in Cowork, with the plugin, and Node.js installed from
  [nodejs.org](https://nodejs.org). On both platforms we confirmed that the
  plugin fails without Node.js and connects once it is installed.
- Sign-in through the consent panel, **Revoke Access**, and reconnecting
  after TouchDesigner or the client restarts.
- **Update** from an earlier release.
- An automated suite of over 900 tests against a running component.

Codex, Cursor, Antigravity and OpenCode use the same server. They are checked
less often than Claude Code and Claude Desktop.

## Not tested yet: tell us what happens

If you run TDMCP in one of these situations, a short report helps, whether it
worked or not.

**Node.js for Claude Desktop**

- Node.js installed with Homebrew, or only through a version manager such as
  nvm, fnm or Volta. Claude Desktop may not find it, because an app does not
  see what your terminal sees. The sign is in
  [The server fails at once](claude-desktop.md#the-server-fails-at-once-nodejs-is-missing).
  On Windows, winget and Chocolatey run the nodejs.org installer, so they
  should behave the same as it.

**Claude accounts**

- A free Claude account. Cowork needs a paid plan; we have not checked what a
  free account is shown.

**Sign-in over time**

- A session left open past 24 hours, when the access token expires.
- Letting the consent panel run out, then trying again without restarting
  Claude Desktop.

**Starting things in a different order**

- The client opened before TouchDesigner, or TouchDesigner closed and
  reopened in the middle of a session.
- **Active** switched off and on while a client is connected.

**Your network and security software**

- A proxy, a VPN, or antivirus that inspects local traffic. One report points
  to a rewritten response stopping the Claude Desktop sign-in
  ([#16](https://github.com/TouchDesigner/TDMCP/issues/16)).

**Scale**

- Long sessions and large projects: thousands of operators, hours of work.

**Updating**

- Moving from the 1.1.57 plugin and component to 1.1.58 on a machine that has
  used both.

## Known limits

- **Claude Desktop needs Node.js 18 or later**, and the plugin works in
  **Cowork** and **Code** sessions, not in plain Chat.
- **The Claude Desktop plugin uses port 13316 only.** A component on another
  port cannot be reached from it.
- **Two components on one port.** The second one does not start, and says
  nothing. Use one component per project, or give the second another
  **Port**.
- **Resetting Ask for authentication to its default turns sign-in off**
  (1.1.56 to 1.1.58). The switch ships on, but its stored default is off.
  Check the switch after resetting parameters on the MCP page. Fixed in the
  next release.

## How to report

Start a [discussion](https://github.com/TouchDesigner/TDMCP/discussions) or
open an [issue](https://github.com/TouchDesigner/TDMCP/issues) with:

- Your TouchDesigner build (**Help → About TouchDesigner**) and the TDMCP
  **Version** from the component's About page.
- Your operating system, and the agent you use with its version.
- For Claude Desktop: how Node.js was installed.
- What you did, what you expected, and what happened.
- The event log if you can: the component's **Storage** page opens the
  folder, and the file is `logs/tdmcp_13316.log`.
