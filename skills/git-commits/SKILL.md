---
name: git-commits
description: "Create safe local atomic commits with focused diffs, short one-line messages, optional gitmoji discipline, and no pushes."
---

# Git Commits

Create atomic local commits.

## Usage

- `/git-commits`
- `/git-commits --scope <scope>`
- `/git-commits --save`
- `/git-commits --fixup <commit>`

## Purpose

Turn working tree changes into focused, reviewable local commits.

## Hard Rules

- NEVER push. Local commits only.
- NEVER commit directly to `main` or `master` unless explicitly authorized.
- Commit message MUST be one short summary line only.
- Do not add a commit body.
- One commit equals one logical unit of change.
- The committed state should build/pass relevant tests when possible.
- If one emoji/type cannot describe the change, split the commit.

## Commit Message Format

Prefer:

```text
<emoji> <type>(<scope>): <message>
```

or plain Conventional Commit if emoji is not requested:

```text
<type>(<scope>): <message>
```

Examples:

```text
✨ feat(auth): add token encryption
fix(config): read default port from env
✅ test(core): add checksum regression test
♻️ refactor(core): extract date validation
⚰️ fix(connector): remove dead code
```

## Save Commits

Use `save(scope): message` for temporary checkpoints. Save commits are allowed during development but should be squashed/reworked before final merge.

## Fixup Commits

Use fixups for review corrections:

```bash
git commit --fixup=<target>
git rebase -i --autosquash <base>
```

Fixups are temporary scaffolding; they should not survive final history unless the workflow explicitly allows it.

## Steps

1. Inspect branch and refuse unsafe branch unless authorized.
2. Inspect working tree.
3. Group changes by logical unit.
4. For each group:
   - show files
   - explain why the group is atomic
   - propose one-line message
5. Stage only that group.
6. Run relevant tests/checks if feasible.
7. Commit locally.
8. Repeat until no requested changes remain.

## Anti-Patterns

- `fix all things`
- `final fix`
- mixing feature + refactor + formatting
- giant PR cleanup bombs
- ambiguous emoji or type
