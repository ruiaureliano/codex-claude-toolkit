# 👨‍🏫 Docs

[![](https://img.shields.io/badge/MIT-License-0f73b4.svg)](./LICENSE.md) [![](https://img.shields.io/badge/shell-bash-0f73b4.svg)](https://www.gnu.org/software/bash/) [![](https://img.shields.io/badge/docs-guide-0f73b4.svg)](./DOCS.md)

`review` is built on top of two CLIs:

1. **Codex CLI** - https://github.com/openai/codex
1. **Claude Code CLI** - https://github.com/anthropics/claude-code

---

#### ⚠️ Please make sure your setup is complete (see how [here](./README.md))

**1)** Every field in `.run/.review.conf` controls one part of the run:

| Field                                                                                   | Controls                                                                                      |
| --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `GIT_PATH`                                                                              | Where `review` looks for repository metadata                                                  |
| `SWIFTLINT_CONFIG_PATH`, `SWIFT_FORMAT_CONFIG_PATH`, `GENERAL_SWIFT_FORMAT_CONFIG_PATH` | Which config SwiftLint/SwiftFormat use, falling back to your home directory's `.swift-format` |
| `XCODE_PROJECT_PATH`, `XCODE_CONFIGURATION`                                             | Which `.xcodeproj` and build configuration an optional build/test pass uses                   |
| `REVIEW_REPORT_DIRECTORY`, `REVIEW_REPORT_PREFIX`                                       | Where timestamped reports are saved, and their filename prefix                                |
| `REVIEW_TIMEOUT_SECONDS`                                                                | How long any single model or Xcode call may run before it's killed                            |
| `STANDARD_RUN_*`, `STANDARD_APPLY_SWIFTFORMAT`                                          | Which steps Standard review runs without asking                                               |
| `STANDARD_AI_SCOPE`                                                                     | `swift` sends only `.swift` files to the models; anything else sends the full selected scope  |
| `STANDARD_AI_REVIEWER`                                                                  | Which model — `codex` or `claude` — performs the final synthesis in Standard review           |
| `STANDARD_REVIEW_SCOPE`                                                                 | `changes` reviews only Git changes; `all` reviews the whole repository                        |

**2)** **Standard review** prints these defaults before doing anything else, so you always see exactly what's about to run:

```
Standard review defaults:
  Review scope: Git changes
  ShellCheck: skipped
  SwiftLint: skipped
  SwiftFormat: skipped
  Xcode build: skipped
  Xcode tests: skipped
  AI review: enabled (Codex + Claude; final: codex, swift files)
```

**3)** **Customize review** asks the same questions Standard review answers for you, one at a time — Run ShellCheck? All files or Git changes? Run SwiftLint? Apply SwiftFormat? Run an Xcode build, then tests? Run the AI review, and if so, which model performs the final synthesis, and what scope goes to the models? Every prompt's default matches `.review.conf`'s Standard answer, so pressing Enter through all of them reproduces Standard review exactly 👍

**4)** Codex and Claude must each return output matching an exact grammar, or that reviewer's run is treated as failed rather than trusted:

```
🔴 [HIGH] path/to/file.swift:123 - finding
🟠 [MEDIUM] path/to/file.swift:45 - finding
🟡 [LOW] path/to/file.swift:8 - finding
```

or exactly `🟢 No issues found.` for a clean review. The final synthesis adds one more required tag right after the severity — `[BOTH]`, `[CODEX]`, or `[CLAUDE]` — and is validated the same strict way before it's trusted.

**5)** If both independent reviews come back clean, `review` skips cross-review and the final synthesis entirely — there's nothing to cross-check — and saves a short "no issues found" report set instead of running two more model calls for nothing 💪

**6)** A full run with findings saves four files to `REVIEW_REPORT_DIRECTORY`:

| File                                | Contents                                                                                                               |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `review-<timestamp>.txt`            | The combined report: validation results, files sent, the merged findings, and the shared/Codex-only/Claude-only counts |
| `review-<timestamp>-codex.txt`      | Codex's own findings — its independent review, or its response after cross-checking Claude                             |
| `review-<timestamp>-claude.txt`     | Claude's own findings, the same way                                                                                    |
| `review-<timestamp>-comparison.txt` | Just the merged, tagged findings list                                                                                  |

**7)** Troubleshooting:

- `Codex CLI is missing.` / `Claude Code CLI is missing.` — both are required before any review runs; the message includes the install command for whichever one is missing.
- `review configuration file is missing.` — create `.run/.review.conf` next to the script, or copy one from another project you've already configured.
- `Codex review failed.` / `Claude review failed.` — prints the first 20 lines of that model's raw output so you can see why it didn't match the required format.

---

I'm [Rui Aureliano](http://ruiaureliano.com), iOS and macOS Engineer at [Olá Brothers](https://theolabrothers.com). We make [Sip](https://sipapp.io) 🤓

[Linkedin](https://www.linkedin.com/in/ruiaureliano) | [Twitter](https://twitter.com/ruiaureliano) | [Github](https://github.com/ruiaureliano) | [Stackoverflow](https://stackoverflow.com/users/881095/ruiaureliano)
