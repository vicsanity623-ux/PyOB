# PyOB — Complete Technical Documentation

> **Version**: 0.3.1 · **Last Updated**: March 2026
> **Architecture**: Python 3.12+ · Mixin-Based Package · GitHub Marketplace Action

---

## Table of Contents

1. [System Philosophy](#1-system-philosophy-constrained-surgical-autonomy)
2. [Architecture Overview](#2-architecture-overview)
3. [Module Reference](#3-module-reference)
   - 3.1 [pyob_launcher.py — Entry Point](#31-pyob_launcherpy--main-cli-entry-point)
   - 3.2 [entrance.py — The Entrance Controller](#32-entrancepy--the-entrance-controller)
   - 3.3 [autoreviewer.py — The Pipeline Orchestrator](#33-autoreviewerpy--the-auto-reviewer)
   - 3.4 [reviewer_mixins.py — Implementation Muscles](#34-reviewer_mixinspy--engine-implementations)
   - 3.5 [pyob_code_parser.py — Structural Analysis](#35-pyob_code_parserpy--structural-analysis)
   - 3.6 [pyob_dashboard.py — SOTA Architect HUD](#36-pyob_dashboardpy--the-architect-hud)
   - 3.7 [core_utils.py — Cloud-Aware Foundation](#37-core_utilspy--core-utilities-mixin)
4. [The Verification & Healing Pipeline](#4-the-verification--healing-pipeline)
5. [Symbolic Dependency Management](#5-symbolic-dependency-management)
6. [The XML Edit Engine](#6-the-xml-edit-engine)
7. [The GitHub Librarian](#7-the-github-librarian-integration)
8. [Headless & Cloud Autonomy](#8-headless--cloud-autonomy)
9. [LLM Backend & Smart Sleep Backoff](#9-llm-backend--resilience)
10. [Persistence & State Vault (.pyob/)](#10-persistence--state-management)
11. [Safety & Rollback Mechanisms](#11-safety--rollback-mechanisms)
12. [Marketplace & Docker Infrastructure](#12-marketplace--docker-infrastructure)
13. [Internal Constants & Rulesets](#13-internal-constants--defaults)
14. [Operational Workflow](#14-operational-workflow)
15. [Troubleshooting](#15-troubleshooting)

---

## 1. System Philosophy: Constrained Surgical Autonomy

PyOB is an autonomous agent built on **constrained agency**. Unlike chat-based assistants that require constant prompting, PyOB is a self-driven engine that operates within a strict "Safety Cage" defined by:

1. **Surgical Patching** — Patches are applied via `<SEARCH>/<REPLACE>` blocks limited to 2-5 line anchors.
2. **Atomic Commits** — Changes are isolated in unique Git branches and submitted as PRs via the Librarian.
3. **Multi-Step Verification** — Every edit must pass a 5-layer gate (XML match → Linter → Mypy → PIR → Smoke Test).
4. **Self-Evolution** — The engine is recursive; it can identify its own logic flaws and refactor its source code.

---

## 2. Architecture Overview

### Modular Package Structure
PyOB has transitioned from a script collection to a standardized Python package located in `src/pyob/`.

```text
CoreUtilsMixin (core_utils.py)
├── Provides: Smart Sleep, Headless Approval, XML Engine, LLM Streaming
│
PromptsAndMemoryMixin (prompts_and_memory.py)
├── Provides: Rule-based Templates (Rule 7: No src. imports), CRUD Memory
│
ValidationMixin + FeatureOperationsMixin (reviewer_mixins.py)
├── Provides: Ruff/Mypy validation, Runtime Auto-Heal, XML Implementation
│
AutoReviewer(All Mixins) (autoreviewer.py)
├── Provides: 6-Phase orchestrator logic
│
EntranceController (entrance.py)
├── Provides: Master loop, Remote Sync, Librarian PR logic, Reboot Flag
```

### System Data Flow (Cloud Service Mode)
```
[User / Schedule] ─▶ [GitHub Action] ─▶ [Docker Container]
                                              │
                    ┌─────────────────────────┴────────────────────────┐
                    │      ENTRANCE CONTROLLER (Master Loop)           │
                    │ 1. Sync Remote Main  2. Pick Target  3. Backup   │
                    └───────────┬────────────────────────────┬─────────┘
                                ▼                            ▼
                    ┌──────────────────────┐      ┌────────────────────┐
                    │   AUTO REVIEWER      │      │    LIBRARIAN       │
                    │ (6-Phase Pipeline)   │      │ (Branch/Commit/PR) │
                    └───────────┬──────────┘      └────────────────────┘
                                ▼
                    ┌──────────────────────────────────────────────────┐
                    │           VERIFICATION & HEALING                 │
                    │ [Ruff --fix] ─▶ [Mypy] ─▶ [10s Smoke Test]      │
                    └──────────────────────────────────────────────────┘
```

---

## 3. Module Reference

### 3.1 `pyob_launcher.py` — Main CLI Entry Point
The environment bootstrapper. It configures the runtime, handles macOS terminal re-launching, and detects "Headless" environments.

| Method | Description |
|---|---|
| `load_config` | Pulls keys from `~/.pyob_config` (Local) or `os.environ` (Cloud). Detects non-TTY to skip prompts. |
| `ensure_terminal` | macOS-specific logic to force PyOB into a visible Terminal window for DMG users. |
| `main` | Entry point. Detects macOS app bundle paths and ignores them to ensure clean targeting. |

### 3.2 `entrance.py` — The Entrance Controller
The master orchestrator. Manages symbolic targeting, Git lifecycle, and Hot-Reboots.

| Method | Description |
|---|---|
| `run_master_loop` | Infinite loop with `sync_with_remote` check. Manages the `self_evolved_flag`. |
| `sync_with_remote` | Fetches `origin/main`. If behind, performs a merge. Triggers reboot if engine files change. |
| `handle_git_librarian` | Creates branch `pyob-evolution-vX-timestamp`, commits as `pyob-bot`, and opens PR. |
| `reboot_pyob` | **Verified Hot-Reboot:** Tests if new code is importable before calling `os.execv` to restart. |

### 3.3 `autoreviewer.py` — The Auto Reviewer
The high-level pipeline orchestrator. Ties together the specialized mixins into the 6-phase autonomous cycle.

### 3.4 `reviewer_mixins.py` — Engine Implementations
Separates "Muscle" from "Brain."

- **`ValidationMixin`**: Runs `ruff format`, then `ruff check --fix`. If errors remain, it triggers the PIR loop.
- **`FeatureOperationsMixin`**: The heavy-duty XML matcher. Interprets AI proposals and writes them to `PEER_REVIEW.md`.

### 3.5 `pyob_code_parser.py` — Structural Analysis
A high-fidelity analysis tool that uses **AST (Python)** and **Regex (JS/CSS)** to map the project architecture. It generates the `<details>` dropdowns seen in `ANALYSIS.md`.

### 3.6 `pyob_dashboard.py` — The Architect HUD
A `BaseHTTPRequestHandler` that serves the SOTA Cyberpunk HUD. Features glassmorphism, responsive mobile layout, and real-time AJAX stats updates.

---

## 4. The Verification & Healing Pipeline

PyOB follows a "Proactive Defense" model to ensure code stability.

### Layer 1: Atomic XML Match
Edits are binary: either every block in a response matches perfectly, or the entire iteration is discarded.

### Layer 2: Syntactic "Broom"
1. **`ruff format`**: Normalizes all whitespace.
2. **`ruff check --fix`**: Automatically clears unused imports and variables without costing AI tokens.
3. **Remaining Errors**: Grouped by file and fed into the AI for surgical repair.

### Layer 3: Runtime Smoke Test
- Locates the entry point via `_find_entry_file`.
- Launches the process for 10 seconds.
- **Auto-Dependency Locking**: If a `ModuleNotFoundError` is detected, PyOB runs `pip install` and immediately updates `requirements.txt`.

---

## 5. Symbolic Dependency Management

### `SYMBOLS.json` Ledger
PyOB maintains a mapping of **Definitions** (where a function/class is born) to **References** (where it is used).

### Symbolic Ripple Engine
1. When a file is edited, the engine identifies changed symbols.
2. It looks up the ledger to see if those symbols are "Definitions."
3. It finds all "References" in other files.
4. **Cascade Queue:** Impacted files are prioritized for the next iteration, with the original change-diff provided as mandatory context.

---

## 6. The XML Edit Engine

### Multi-Strategy Matching
`apply_xml_edits` attempts 5 strategies per block:
1. **Exact** (Literal)
2. **Stripped** (Newline tolerance)
3. **Normalized** (Comment/Whitespace stripping)
4. **Regex Fuzzy** (Indentation tolerance)
5. **Robust Line Match** (Content-only line comparison)

### Smart Indent Alignment
The engine detects the target line's indentation and re-aligns the AI's `<REPLACE>` block to match, preventing the "Unexpected Indentation" errors common in Python agents.

---

## 7. The GitHub Librarian Integration

PyOB acts as a professional developer through the **Librarian** module:

- **Isolated Branches:** Every change is pushed to a unique branch.
- **Bot Identity:** Commits are attributed to `pyob-bot` using the `BOT_GITHUB_TOKEN`.
- **Automated PRs:** Uses the GitHub CLI (`gh`) to open Pull Requests targeting `main`.
- **PR Body:** Includes the AI's `<THOUGHT>` process as the PR description for human review.

---

## 8. Headless & Cloud Autonomy

PyOB detects when it is running in **GitHub Actions** (via the `GITHUB_ACTIONS=true` env var):

- **Auto-Approval:** Bypasses "Press ENTER to apply" prompts.
- **Non-TTY Safety:** Skips all `termios` and `input()` calls to prevent `EOFError` or `ioctl` crashes.
- **Cloud Tunneling:** Starts a background **Pinggy** tunnel to provide a public URL for the dashboard HUD.

---

## 9. LLM Backend & Smart Sleep Backoff

### Multi-Key Key Rotation
PyOB rotates through a pool of up to 10 Gemini API keys. Keys that hit a `429 Rate Limit` are quarantined for 20 minutes.

### Smart Sleep Backoff
When all keys are rate-limited, the engine calculates:
`sleep_duration = min(key_cooldowns) - current_time`
The bot "naps" for the exact number of seconds until the first key is available, ensuring zero waste of Cloud Runner minutes.

---

## 10. Persistence & State Management (.pyob/)

All project metadata is stored in the hidden `.pyob/` vault to prevent root directory clutter.

| File | Purpose |
|---|---|
| `.pyob/ANALYSIS.md` | Persistent map used by the AI to select targets. |
| `.pyob/SYMBOLS.json` | The symbolic dependency graph. |
| `.pyob/MEMORY.md` | Transactional AI memory; refactored every 2 iterations to prevent context bloat. |
| `.pyob/HISTORY.md` | Detailed ledger of applied patches. |

---

## 11. Safety & Rollback Mechanisms

- **External Safety Pods:** Before editing an "Engine File" (like `entrance.py`), PyOB shelters a copy of the current source in `~/Documents/PYOB_Backups/`.
- **Workspace Backup:** Every iteration starts with an in-memory snapshot of the entire project.
- **Atomic Rollback:** If any verification layer (Linter, Mypy, or Runtime) fails 3 times, the entire workspace is restored to the backup.

---

## 12. Marketplace & Docker Infrastructure

### Marketplace Action
PyOB is a containerized GitHub Action (`action.yml`). It uses a `Dockerfile` based on `python:3.12-slim` with `git`, `curl`, and `gh` pre-installed.

### Docker Environment
The Docker container maps the user's repository to `/github/workspace`, allowing PyOB to operate on the files as if it were a local CLI tool.

---

## 13. Internal Constants & Rulesets

### Mandatory Import Rule (Rule 7)
The AI is strictly prohibited from using the `src.` prefix in imports.
- **Correct:** `from pyob.core_utils import ...`
- **Incorrect:** `from src.pyob.core_utils import ...`

### Indentation Guard (Rule 6)
Deletions must leave a placeholder comment (e.g., `# [Logic moved to new module]`) to maintain Python's indentation integrity.

---

## 14. Operational Workflow

1. **Remote Sync:** Pull latest merges from GitHub.
2. **Genesis / Update:** Build or refresh `ANALYSIS.md` and `SYMBOLS.json`.
3. **Targeting:** Select file via AI or the `Cascade Queue`.
4. **Pipeline:** Scan → Propose → Verify → Auto-Heal.
5. **Librarian:** Push Branch → Open PR.
6. **Self-Evolution:** If engine changed, verify importability and Hot-Reboot.

---

## 15. Troubleshooting

### `ModuleNotFoundError: No module named 'src'`
**Cause:** AI incorrectly added `src.` to an import statement.
**Fix:** Remove the `src.` prefix. Ensure `pyproject.toml` is installed via `pip install -e .`.

### `EOFError: EOF when reading a line`
**Cause:** PyOB tried to call `input()` in a cloud environment.
**Fix:** Ensure `sys.stdin.isatty()` checks are present in the launcher.

### `termios.error: Inappropriate ioctl for device`
**Cause:** `get_user_approval` tried to manipulate a non-existent keyboard.
**Fix:** The "Headless Auto-Approval" logic in `core_utils.py` handles this.

---
> **PyOB** — The engine that builds itself, with surgical precision. 🦅
