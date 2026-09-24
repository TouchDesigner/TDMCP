# OpenCode

[OpenCode](https://opencode.ai) is an open-source agent CLI that can drive any model provider, including a model running on your own machine. That makes it the most practical way to try TDMCP with a **local agent**: no API key, no network, no per-token cost.

It is also the least conventional client TDMCP supports. Its config shape, its tool plumbing and its skills discovery all differ from the other clients, and a default setup quietly defeats small models. This page collects what has been established by probing, and what is still open.

## Two versions

TDMCP supports both, as separate entries in **Install For**:

| | OpenCode (v1) | OpenCode v2 (beta) |
| --- | --- | --- |
| Binary | `opencode` | `opencode2` during the beta |
| Verified | 1.17.7, 1.18.30 | 2.0.16 |
| Skills | none located, so connection only | read from six directories |
| `mcp add` writes | the global config only | the project `./opencode.json`, or global with `--global` |
| Config shape | `mcp.<name>` | `mcp.servers.<name>` |
| Tools reach the model | directly (not tested with a local model) | through Code Mode by default; see [Code Mode](#code-mode) |

Both read `~/.config/opencode/`, and v1 (1.18.30) also reads v2's nested shape, so one global config serves both side by side. v1 is the stable release and the one most people have. v2 is where the local-agent story is strongest, and the rest of this page is mostly about it. Check the [v2 docs](https://dev.opencode.ai/v2/docs/) for its current status; when it leaves beta, the `opencode` entry will move to v2.

## Quick start: a local agent

This is the tested setup: OpenCode v2 beta, Ollama and `qwen3-coder:30b` on an M1 Max with 64 GB.

### 1. Install OpenCode v2

```bash
npm install -g @opencode-ai/cli@next   # the v2 beta, per OpenCode's v2 docs
opencode2 --version                    # must print 2.x
```

During the beta the binary is `opencode2`, so it sits beside a v1 `opencode` from Homebrew or the standard installer. Some installs also put a v2 binary named `opencode` in `~/.opencode/bin`, which then shadows v1 in interactive shells. `type -a opencode opencode2` shows which binary each name runs. PATH additions in `~/.zshrc` apply only to interactive terminals, not to apps launched from the Dock.

### 2. Run a model

OpenCode is only the agent. It does not bundle or start a model, so something else must be serving one, or every prompt fails with `ConnectionRefused`. Ollama is the least effort: it runs as a background service and loads the model on the first request.

```bash
brew install ollama
brew services start ollama          # starts now and at every login
ollama pull qwen3-coder:30b         # ~18 GB, 4-bit
```

Raise the context window. Ollama's default is far too small for TDMCP, whose tool definitions and server instructions alone fill several thousand tokens. A derived model shares the weights, so it costs no extra disk:

```bash
printf 'FROM qwen3-coder:30b\nPARAMETER num_ctx 65536\n' > Modelfile
ollama create qwen3-coder-64k -f Modelfile
```

At 64K context it uses about 22 GB of memory, fully on the GPU.

### 3. Configure OpenCode

Put this in `opencode.json` in your project folder, or in `~/.config/opencode/opencode.json` to apply it everywhere:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/qwen3-coder-64k",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": { "baseURL": "http://localhost:11434/v1" },
      "models": { "qwen3-coder-64k": { "name": "Qwen3-Coder 30B, 64K" } }
    }
  },
  "mcp": {
    "servers": {
      "touchdesigner": {
        "type": "remote",
        "url": "http://127.0.0.1:13316/mcp",
        "codemode": false
      }
    }
  }
}
```

Match the port and server name to the component's **Port** and **Servername**. `"codemode": false` matters most; see [Code Mode](#code-mode) below.

### 4. Install the skills

In the TDMCP component, set **Install For** to OpenCode v2 (beta), choose a scope and pulse **Install Agent Skills**. Project scope writes `<project>/.agents/skills/`, the same folder Codex and Antigravity read, so one install serves all three.

### 5. Check it

```bash
opencode2 mcp list                  # touchdesigner  ✓ connected
opencode2 debug config | grep -A4 '"touchdesigner"'   # codemode: false must survive
```

Then run `opencode2` in the project folder.

## How OpenCode differs

### Code Mode

This is the setting that decides whether a small model gets anywhere.

By default, v2 does not give the model TDMCP's tools. It gives the model one tool, `execute`, which runs JavaScript, and exposes the MCP tools inside that sandbox as functions like `tools["touchdesigner"].get_help({types: [...]})`. So the model has to write valid JavaScript *and* get each tool's arguments right.

Strong hosted models cope with this. `qwen3-coder:30b` did not. It wrote Python-style calls such as `list_operators(path="/project1")`, which fail in JavaScript, retried about twenty times and never created an operator. With `"codemode": false` on the server entry, the same model called the tools directly and built a working network.

`opencode2 mcp add` has no flag for this, so add the key by hand.

### Config shape

| | v2 |
| --- | --- |
| Container | `mcp.servers.<name>` — not `mcpServers`, and not v1's flat `mcp.<name>` |
| Entry | `{"type": "remote", "url": "…", "codemode": false}` |
| Project file | `./opencode.json`, merged over the global one |
| Global file | `~/.config/opencode/opencode.jsonc` if it exists, else `opencode.json` |

v2 still reads the v1 flat shape, but it migrates it on load and **drops keys it does not carry over**, `codemode` among them. The file looks right and the setting has no effect. `opencode2 debug config` shows what v2 actually resolved; trust that, not the file.

### Registration

```bash
opencode2 mcp add touchdesigner --url http://127.0.0.1:13316/mcp            # ./opencode.json
opencode2 mcp add touchdesigner --url http://127.0.0.1:13316/mcp --global   # user config
```

v2 writes the project file by default, unlike v1. There is no `mcp remove`: delete the entry from the file. The component's **Remove MCP** field says which file.

### Skills

v2 discovers skills in six places, all confirmed with marker skills:

| Scope | Paths |
| --- | --- |
| Project | `.opencode/skills/`, `.agents/skills/`, `.claude/skills/` |
| Global | `~/.config/opencode/skills/`, `~/.agents/skills/`, `~/.claude/skills/` |

So an existing Claude Code or Codex install is already visible to OpenCode. Duplicates resolve by skill name. The component installs to `~/.config/opencode/skills/` or `.agents/skills/`.

To see what OpenCode found without spending a model call:

```bash
opencode2 api GET /api/skill -H "x-opencode-directory:$PWD"
```

### The background service

`opencode2 run` and the TUI connect to a background server, which holds the config it loaded. After editing a config file, run `opencode2 reload`.

## Findings so far

The same task in both runs: *build an animated, colourful abstract texture inside a new baseCOMP, ending in a null TOP, and check for errors.* Model `qwen3-coder:30b` (Q4_K_M, 64K context) through Ollama on an M1 Max with 64 GB, with the TDMCP skills installed at project scope.

| | Code Mode on (default) | Code Mode off |
| --- | --- | --- |
| Time | 6 min, gave up | 20 min, finished |
| Operators created | none | a noise → ramp → null chain, no errors |
| Tool calls | ~20 failed `execute` attempts | direct calls to `get_help`, `build_network`, `set_parameters`, `wiring` |
| Outcome | wrote three how-to files to disk for the user instead | a static image, no animation |

With Code Mode off, the model did several things right. It loaded skills unprompted, called `get_help` before setting parameters, recovered when a parameter name was wrong, and read its own errors.

It still:

- ignored rules stated in both the skills and the server instructions: it used `absTime.seconds` and absolute `/project1/...` paths
- tried to wire a CHOP into a TOP input, then deleted and rebuilt the whole component three times instead of fixing one connection
- sent a JSON *string* where a tool expected an object, which the client rejected before the call reached TouchDesigner
- wrote a GLSL shader with its own `uTime` uniform, which TouchDesigner never sets, then abandoned it
- saved the project file without being asked

Taken together: with Code Mode off, a 30B local model can drive the MCP, but not yet to a finished result. The protocol side works. What fails is TouchDesigner knowledge and follow-through.

## Open questions

These are worth investigating, and results are welcome:

- **Other local models.** Gemma 4 26B-A4B is faster on Apple silicon and can read images, which suits `view_operator`. Dense models such as Qwen3.8-27B may follow instructions better but run several times slower. The same task across models would show where the quality threshold is.
- **Instructions that land.** The model loaded skills but ignored their hard rules. Does a short project `AGENTS.md` with the half-dozen rules that matter do better than full skills for a small model, or worse?
- **Context length.** Does 32K lose anything against 64K? Does 128K help long builds?
- **A smaller tool surface.** OpenCode can deny tools per agent. Does hiding `execute_code` push a small model toward the dedicated tools?
- **A hosted model through OpenCode.** The same task with a frontier model would separate "OpenCode setup" from "model capability" cleanly.
- **Speed.** Reading TDMCP's tool definitions at the start of a session is the slow part on Apple silicon, so it deserves measuring on its own.

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| `AI.Error: ConnectionRefused` on every prompt | No model is being served at the provider's `baseURL`. Start Ollama (`brew services start ollama`) or your MLX/LM Studio server. |
| The model calls `execute` and writes JavaScript | Code Mode is on. Add `"codemode": false`, run `opencode2 reload`, and check with `opencode2 debug config`. |
| `codemode` is in the file but has no effect | The entry is in the v1 flat shape and was migrated. Move it under `mcp.servers`. |
| `Invalid arguments … Expected object` | The model sent a string where an object was expected. That is a model error, and the client rejects it before TouchDesigner sees it. |
| `opencode2` is not found, or `opencode --version` prints 2.x | The v2 install put its binary under another name or path. Check `type -a opencode opencode2`. |
| Config edits are ignored | The background service still has the old config. Run `opencode2 reload`. |
