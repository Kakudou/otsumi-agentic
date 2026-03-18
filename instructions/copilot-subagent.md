# Copilot CLI: SubAgent Delegation

---

## Delegating to SubAgents

CRITICAL: Before engaging in any stage-specific logic, or executing a skill you must delegate to the corresponding subagent using `task` tools. This ensures that all stage processing is encapsulated within its dedicated agent, maintaining clear separation of concerns and adhering to the pipeline architecture.

Instructions:

- use runSubagent to execute the process of ~ in a subagent, and notify the invocation.
- If a subagent with the same name is already spawned, reuse it instead of creating a new one.
- use read_agent to show the subagent output
- when all tasks are done, kill the subagent
