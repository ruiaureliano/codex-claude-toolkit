# 🤝 Codex Claude Toolkit

[![](https://img.shields.io/badge/MIT-License-0f73b4.svg)](./LICENSE.md) [![](https://img.shields.io/badge/shell-bash-0f73b4.svg)](https://www.gnu.org/software/bash/) [![](https://img.shields.io/badge/docs-guide-0f73b4.svg)](./DOCS.md)

`review` is a shell script that reviews a Git repository with **Codex** and **Claude Code** as two independent, read-only reviewers, has each one cross-check the other's findings, then merges everything into a single severity-tagged list.

## Requirements

- [Codex CLI](https://github.com/openai/codex) — `curl -fsSL https://chatgpt.com/codex/install.sh | sh`
- [Claude Code CLI](https://github.com/anthropics/claude-code) — `curl -fsSL https://claude.ai/install.sh | bash`
- `git`. For the optional local checks: `shellcheck`, `swiftlint`, `swift-format`, `xcodebuild` — each one is skipped quietly if it isn't installed 👍

## Installation

### Using Github

```
git clone https://github.com/ruiaureliano/codex-claude-toolkit.git
chmod +x codex-claude-toolkit/.run/review
```

Copy `.run/review` into your project's `.run/` folder, then create `.run/.review.conf` next to it:

```bash
GIT_PATH="$REPOSITORY_PATH/.git"
SWIFTLINT_CONFIG_PATH="$REPOSITORY_PATH/.swiftlint.yml"
SWIFT_FORMAT_CONFIG_PATH="$REPOSITORY_PATH/.swift-format"
GENERAL_SWIFT_FORMAT_CONFIG_PATH="$HOME/.swift-format"
XCODE_PROJECT_PATH="$REPOSITORY_PATH/YourApp.xcodeproj"
XCODE_SCHEME=""
XCODE_CONFIGURATION="Debug"
REVIEW_REPORT_DIRECTORY="$HOME/Desktop"
REVIEW_REPORT_PREFIX="review"
REVIEW_TIMEOUT_SECONDS=300
REVIEW_SCRIPT_PATH="$REPOSITORY_PATH/.run/review"

# Answers used by the Standard review mode.
STANDARD_RUN_SHELLCHECK=0
STANDARD_RUN_SWIFTLINT=0
STANDARD_APPLY_SWIFTFORMAT=0
STANDARD_RUN_XCODE_BUILD=0
STANDARD_RUN_XCODE_TESTS=0
STANDARD_RUN_AI_REVIEW=1
STANDARD_AI_SCOPE='swift'
STANDARD_AI_REVIEWER='codex'
STANDARD_REVIEW_SCOPE='changes'
```

`review` refuses to run without this file — it's project-local configuration, never bundled defaults, so it's clear exactly which paths and tools a given project's review will touch 🖥

## Usage

### 1) Pick a review mode

```
./review
```

```
How would you like to run the review?
❯ Standard review (recommended)
  Customize review
```

**Standard review** runs `.review.conf`'s fixed answers with no further questions and prints them up front. **Customize review** asks about scope, SwiftLint, SwiftFormat, an Xcode build and tests, and which model does the final synthesis, one question at a time.

### 2) Local checks run first

```
☑ 3 changed files
• Sources/Sip/Brewer.swift
• Sources/Sip/Timer.swift
+ Tests/BrewerTests.swift
```

ShellCheck lints the script itself, SwiftLint prints one icon per file (❌ errors, ⚠️ warnings, ✅ clean) instead of raw diagnostics, and SwiftFormat's prompt — the only step that rewrites the working tree — defaults to **No** even though every other prompt defaults to **Yes** 💪

### 3) Codex and Claude review in parallel

Both models get an identical prompt: the same instructions, the same one-finding-per-line severity format, and the same `git diff`. Output that doesn't match the required grammar is rejected and retried as a failure rather than trusted as-is.

If both come back clean, cross-review and synthesis are skipped entirely and a short "no issues" report is saved. Otherwise, each model re-checks the other's raw findings against the code and confirms, rejects, or adds to them:

```
Claude has completed an independent review below.
Now verify Claude findings against the code, confirm or reject them, and add
any missing findings.
Codex independent findings:
...
```

### 4) One model synthesizes the result

You choose which model — Codex or Claude — merges both cross-checked lists into one, tagging every line with who raised it:

```
🔴 [HIGH] [BOTH] Sources/Sip/Brewer.swift:123 - ignores the cancellation token
🟠 [MEDIUM] [CODEX] Sources/Sip/Timer.swift:45 - timer leaks on early return
```

Any `[BOTH]` finding, or any `HIGH` severity finding from either model, fails the run — a disagreement between two models isn't automatically wrong, but it's a signal worth reading before shipping 🚀

### 5) Reports are saved automatically

Every run writes timestamped files to `REVIEW_REPORT_DIRECTORY` — the combined summary plus separate Codex, Claude, and comparison reports — so a review from an hour ago is still there to compare against.

---

I'm [Rui Aureliano](http://ruiaureliano.com), iOS and macOS Engineer at [Olá Brothers](https://theolabrothers.com). We make [Sip](https://sipapp.io) 🤓

[Linkedin](https://www.linkedin.com/in/ruiaureliano) | [Twitter](https://twitter.com/ruiaureliano) | [Github](https://github.com/ruiaureliano) | [Stackoverflow](https://stackoverflow.com/users/881095/ruiaureliano)
