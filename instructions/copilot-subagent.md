# Copilot CLI: Subagent Delegation (Otsumi)

Use the real Copilot toolchain for delegation:

- Invoke specialists with the `task` tool.
- Read outputs with `read_agent`.
- Do not reference `runSubagent`, `kill`, or any nonexistent subagent command.

Only these shogi agents are on-board:

- `osho`, `kakugyo`, `kinsho`, `ginsho`, `hisha`, `kyosha`, `fuhyo`, `keima`

Off-board prohibition:

- NEVER use `general-purpose`, bare `Task`, `claude-code`, `agent`, `subagent`, or any non-shogi agent.

Universal handoff envelope (return shape):

```json
{
  "task_completed": true,
  "blocked": false,
  "blocker": null,
  "agent_output": {}
}
```

Blocked form:

```json
{
  "task_completed": false,
  "blocked": true,
  "blocker": {
    "reason": "enum value",
    "detail": "actionable explanation",
    "agent": "agent-name"
  },
  "agent_output": {}
}
```
