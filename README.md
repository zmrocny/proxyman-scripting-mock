# proxyman-scripting-mock

An [agent skill](https://agentskills.io) for backend mocks in [Proxyman](https://proxyman.com) Scripting: the agent writes a mock script to disk and mirrors it as a Scripting rule in Proxyman.

- **Source of truth:** a `.js` file in `~/Proxyman/Scripting/[<Company>/]<Product>/[Testing/]`; the Proxyman rule mirrors it.
- **Modes:** mock API when the backend is down (no upstream, CORS preflight answered), or response override when the backend is up.
- **Create, update, disable:** to update, the agent edits the file first, then copies it into the Proxyman rule. To disable, it turns the rule off and keeps both file and rule. It deletes only when you ask.

See [`skills/proxyman-scripting-mock/SKILL.md`](skills/proxyman-scripting-mock/SKILL.md) for the full rules.

## Requirements

- Proxyman (macOS, Windows or Linux), running, with MCP enabled.
- The Proxyman MCP server connected to your agent.

## Install

### Any agent (skills CLI)

Works with Claude Code, Codex, Cursor, OpenCode and other agents that support Agent Skills:

```sh
npx skills add zmrocny/proxyman-scripting-mock
```

The CLI detects installed agents and asks where to put the skill.

### Any agent (APM)

With [APM](https://github.com/microsoft/apm), the skill is recorded in your project's `apm.yml` and pinned in `apm.lock.yaml`:

```sh
apm install zmrocny/proxyman-scripting-mock
```

APM detects agents from the project folders. To choose them yourself, add `--target`, for example `--target claude,cursor`.

### Claude Code (plugin)

```
/plugin marketplace add zmrocny/proxyman-scripting-mock
/plugin install proxyman-scripting-mock@proxyman-scripting-mock
```

### Manually

Copy `skills/proxyman-scripting-mock/` into your agent's skills directory, for example `~/.claude/skills/` or `.agents/skills/`.

## License

[MIT](LICENSE)
