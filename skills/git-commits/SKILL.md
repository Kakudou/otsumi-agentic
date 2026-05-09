---
name: git-commits
description: "Create safe local atomic commits with focused diffs, short one-line messages, gitmoji discipline, and no pushes."
---

# Git Commits

Turn working-tree changes into focused, reviewable local commits.

## Usage

- `/git-commits`
- `/git-commits --scope {scope}`
- `/git-commits --save`
- `/git-commits --fixup {commit}`

## Mandatory Routing

Any agent creating a git commit MUST run through this skill. Calling `git commit` via raw bash without going through the discipline below (group → atomicity check → emoji pick → one-line message → stage → commit) is forbidden — it bypasses the rules and produces messy history. If the agent is not authorized for `git-commits`, it returns `blocked` rather than shelling out a raw commit.

## Hard Rules

- ALWAYS ask and wait for user go-to before commiting, giving time to review/edit files before.
- NEVER push. Local commits only.
- NEVER commit directly to `main` or `master` unless explicitly authorized.
- The commit message MUST be one short summary line — no body.
- One commit MUST equal one logical unit of change.
- The committed state SHOULD build / pass relevant tests when possible.
- If one emoji/type cannot describe the change, MUST split the commit.
- Pick the SINGLE dominant emoji for the diff. Two equally-fitting emojis means the commit is too broad — split.

## Commit Message Format

Preferred (with gitmoji):

```text
{emoji} {type}({scope}): {message}
```

Plain Conventional Commit (when the gitmoji set does not cover the intent):

```text
{type}({scope}): {message}
```

Examples:

```text
✨ feat(auth): add token encryption
🐛 fix(config): read default port from env
✅ test(core): add checksum regression test
♻️ refactor(core): extract date validation
⚰️ chore(connector): remove dead code
```

## Gitmoji Reference

Pick the single emoji whose description best matches the dominant intent of the diff.

### Features, releases, milestones

| Emoji | Code | Use when |
|---|---|---|
| ✨ | `:sparkles:` | Introduce a new feature. |
| 💥 | `:boom:` | Introduce breaking changes. |
| 🎉 | `:tada:` | Begin a project. |
| 🔖 | `:bookmark:` | Release / version tag. |
| 🚀 | `:rocket:` | Deploy stuff. |

### Bug fixes & error handling

| Emoji | Code | Use when |
|---|---|---|
| 🐛 | `:bug:` | Fix a bug. |
| 🚑️ | `:ambulance:` | Critical hotfix. |
| 🩹 | `:adhesive_bandage:` | Simple fix for a non-critical issue. |
| 🥅 | `:goal_net:` | Catch errors. |
| ⏪️ | `:rewind:` | Revert changes. |

### Refactor & cleanup

| Emoji | Code | Use when |
|---|---|---|
| 🎨 | `:art:` | Improve structure / format of the code. |
| ♻️ | `:recycle:` | Refactor code. |
| 🔥 | `:fire:` | Remove code or files. |
| ⚰️ | `:coffin:` | Remove dead code. |
| 🗑️ | `:wastebasket:` | Deprecate code that needs cleanup. |
| 🚧 | `:construction:` | Work in progress. |
| 💩 | `:poop:` | Bad code that needs to be improved later. |

### Performance, types, validation, concurrency

| Emoji | Code | Use when |
|---|---|---|
| ⚡️ | `:zap:` | Improve performance. |
| 🏷️ | `:label:` | Add or update types. |
| 🦺 | `:safety_vest:` | Add or update validation code. |
| 🧵 | `:thread:` | Add or update multithreading / concurrency. |

### Tests

| Emoji | Code | Use when |
|---|---|---|
| ✅ | `:white_check_mark:` | Add, update, or pass tests. |
| 🧪 | `:test_tube:` | Add a failing test. |
| 📸 | `:camera_flash:` | Add or update snapshots. |
| 🤡 | `:clown_face:` | Mock things. |

### Documentation, comments, text

| Emoji | Code | Use when |
|---|---|---|
| 📝 | `:memo:` | Add or update documentation. |
| 💡 | `:bulb:` | Add or update comments in source code. |
| 💬 | `:speech_balloon:` | Add or update text and literals. |
| 📄 | `:page_facing_up:` | Add or update license. |

### UI, UX, accessibility, assets

| Emoji | Code | Use when |
|---|---|---|
| 💄 | `:lipstick:` | Add or update UI / style files. |
| 🚸 | `:children_crossing:` | Improve user experience / usability. |
| ♿️ | `:wheelchair:` | Improve accessibility. |
| 📱 | `:iphone:` | Work on responsive design. |
| 💫 | `:dizzy:` | Add or update animations / transitions. |
| 🍱 | `:bento:` | Add or update assets. |

### Dependencies

| Emoji | Code | Use when |
|---|---|---|
| ➕ | `:heavy_plus_sign:` | Add a dependency. |
| ➖ | `:heavy_minus_sign:` | Remove a dependency. |
| ⬆️ | `:arrow_up:` | Upgrade dependencies. |
| ⬇️ | `:arrow_down:` | Downgrade dependencies. |
| 📌 | `:pushpin:` | Pin dependencies to specific versions. |

### Build, CI, config, scripts, packaging

| Emoji | Code | Use when |
|---|---|---|
| 🔧 | `:wrench:` | Add or update configuration files. |
| 🔨 | `:hammer:` | Add or update development scripts. |
| 👷 | `:construction_worker:` | Add or update CI build system. |
| 💚 | `:green_heart:` | Fix CI build. |
| 🚨 | `:rotating_light:` | Fix compiler / linter warnings. |
| 📦️ | `:package:` | Add or update compiled files or packages. |
| 🙈 | `:see_no_evil:` | Add or update `.gitignore`. |

### Security, auth, secrets

| Emoji | Code | Use when |
|---|---|---|
| 🔒️ | `:lock:` | Fix security / privacy issues. |
| 🔐 | `:closed_lock_with_key:` | Add or update secrets. |
| 🛂 | `:passport_control:` | Auth, roles, permissions. |

### Architecture, infrastructure, external interfaces

| Emoji | Code | Use when |
|---|---|---|
| 🏗️ | `:building_construction:` | Make architectural changes. |
| 🧱 | `:bricks:` | Infrastructure-related changes. |
| 🩺 | `:stethoscope:` | Add or update healthcheck. |
| 🚚 | `:truck:` | Move or rename resources (files, paths, routes). |
| 👽️ | `:alien:` | Update code due to external API changes. |
| 🦖 | `:t-rex:` | Code that adds backwards compatibility. |

### Data, analytics, logs

| Emoji | Code | Use when |
|---|---|---|
| 📈 | `:chart_with_upwards_trend:` | Add or update analytics or tracking. |
| 🗃️ | `:card_file_box:` | Database-related changes. |
| 🌱 | `:seedling:` | Add or update seed files. |
| 🧐 | `:monocle_face:` | Data exploration / inspection. |
| 🔊 | `:loud_sound:` | Add or update logs. |
| 🔇 | `:mute:` | Remove logs. |

### Localization, SEO, offline

| Emoji | Code | Use when |
|---|---|---|
| 🌐 | `:globe_with_meridians:` | Internationalization / localization. |
| 🔍️ | `:mag:` | Improve SEO. |
| ✈️ | `:airplane:` | Improve offline support. |

### Project meta & misc

| Emoji | Code | Use when |
|---|---|---|
| ✏️ | `:pencil2:` | Fix typos. |
| 🔀 | `:twisted_rightwards_arrows:` | Merge branches. |
| 👥 | `:busts_in_silhouette:` | Add or update contributors. |
| 👔 | `:necktie:` | Add or update business logic. |
| 🚩 | `:triangular_flag_on_post:` | Add, update, or remove feature flags. |
| ⚗️ | `:alembic:` | Perform experiments. |
| 🥚 | `:egg:` | Add or update an easter egg. |
| 🧑‍💻 | `:technologist:` | Improve developer experience. |
| 💸 | `:money_with_wings:` | Sponsorship / money infrastructure. |
| 🍻 | `:beers:` | Code written drunkenly. |

## Save Commits

Use `save(scope): message` for temporary checkpoints. Save commits are allowed during development but SHOULD be squashed / reworked before final merge.

## Fixup Commits

Use fixups for review corrections:

```bash
git commit --fixup={target}
git rebase -i --autosquash {base}
```

Fixups are temporary scaffolding; they SHOULD NOT survive final history unless the workflow explicitly allows it.

## Steps

1. Inspect branch and refuse unsafe branches unless authorized.
2. Inspect working tree.
3. Group changes by logical unit.
4. For each group:
   - show files
   - explain why the group is atomic (one logical unit, one dominant intent)
   - pick the SINGLE best-matching emoji from the reference table
   - propose a one-line message in the format above
5. Stage only that group.
6. Run relevant tests/checks if feasible.
7. Commit locally.
8. Repeat until no requested changes remain.

## Anti-Patterns

- `fix all things`
- `final fix`
- mixing feature + refactor + formatting in one commit
- giant PR cleanup bombs
- ambiguous emoji or type
- two emojis in one message — means the commit is not atomic, split it
- raw `git commit` shelled out from an agent without running this skill
