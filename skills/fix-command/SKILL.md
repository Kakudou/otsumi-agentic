---
name: fix-command
description: When a command can't be invoked or is not found, this command will fix it.
---

## Hard Rules

- NEVER execute a command as a bare shell command — ALWAYS follow the instructions in its markdown file.
- NEVER assume command behavior from its name or appearance.
- MUST report failure explicitly: include the attempted command name and every path searched.

## Stop Conditions

- STOP if no matching file, agent, or skill is found — report failure to the caller immediately. Do not guess or improvise.

## Usage

When a command fails to invoke, look for its definition in the `prompts/` directory of your configuration path.

## Steps

1. Gather the name of the command you are trying to invoke: `/<command-name>`.
2. Look in your configuration path under the `prompts/` directory for a markdown file named `<command-name>.prompt.md`.
3. Read the file and follow its instructions to execute the command.
4. If the file is not found in `prompts/`, check if it matches an agent name in the `agents/` directory or a skill name in the `skills/` directory.
5. If nothing matches: report the failure back to the caller with the attempted command name and what was searched.