---
name: fix-command
description: When a command can't be invoked or is not found, this command will fix it.
---

Fixes a command that can't be invoked or is not found.

## Usage

When a command fails to invoke, look for its definition in the `prompts/` directory of your configuration path.

## Steps

1. Gather the name of the command you are trying to invoke: `/<command-name>`.
2. Look in your configuration path under the `prompts/` directory for a markdown file named `<command-name>.prompt.md`.
3. Read the file and follow its instructions to execute the command.
4. If the file is not found in `prompts/`, check if it matches an agent name in the `agents/` directory or a skill name in the `skills/` directory.
5. If nothing matches: report the failure back to the caller with the attempted command name and what was searched.

## Hard Rules

- Never try to execute a command as a bare shell command; follow the instructions in the markdown file instead.
- Never make assumptions about the command's behavior based on its name or appearance.
- If you can't find or run the command, refer back to the caller with an explicit report of what was attempted.