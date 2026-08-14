# Flutter Defect Triage

<div align="center">

> **Align the pits first. Then fix.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Cursor](https://img.shields.io/badge/Cursor-Compatible-black)](https://cursor.sh)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

</div>

A defect triage skill for Cursor / Claude Code, built for **Flutter Web, App WebView, mini-program H5, and multi-host JSBridge** — environments where pit density is high and naive fixes routinely fail.

This is **not** a "fix my bug" prompt. It forces the agent to do four things before touching any code:

1. **Select a business pack** — identify the runtime target, gate, and hotspot files
2. **Scan for known pits** — read the RAG index first, then open the matching Case
3. **Pit alignment** — present a mechanism table for your confirmation; ask about anything uncertain
4. **Narrow fix** — write a four-dimension Scope Gate; backfill the Case after merge

```
User reports a defect
    → Select businesses/<slug>/pack.md
    → Search cases-rag-index.md by biz / symptom / file
    → Output pit alignment table (user can correct)
    → Prove mechanism with code + git timeline
    → Minimal diff
    → Backfill cases.md
```

[Installation](#installation) · [Usage](#usage) · [Why Not Just Fix It](#why-not-just-fix-it) · [Repository Structure](#repository-structure) · [Onboarding a Real Project](#onboarding-a-real-project)

---

## Why Not Just Fix It

Flutter Web / WebView defects are rarely "one wrong line of CSS." Common failure modes:

| What you hear | What's actually happening |
|---------------|---------------------------|
| "In the App" | Could be WebView, could be Native; gate is on the wrong side |
| Label present in one place, missing in another | List / bubble / overlay use three different data paths |
| Fixed device A, broke device B | Using magic coefficients to guess host keyboard / safe area |
| Long-press does nothing | An `await` in the gesture chain dropped the pointer |
| Same pit for the third time | Fixed without recording a Case; next agent starts from zero |

This skill turns these into an executable workflow: **no fix code before alignment.**

---

## Installation

### Cursor (Recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/lnuxe/flutter-defect-triage/main/setup | bash
```

Installs to `~/.cursor/skills/flutter-defect-triage/` by default. Start a new conversation after installation.

If you've already cloned the repo locally:

```bash
git clone https://github.com/lnuxe/flutter-defect-triage.git
cd flutter-defect-triage && bash setup
```

### Claude Code

The same `setup` script also writes to `~/.claude/skills/flutter-defect-triage/` when the directory exists or Claude Code is installed.

### Manual

Place this repository at:

| Tool | Path |
|------|------|
| Cursor (global) | `~/.cursor/skills/flutter-defect-triage/` |
| Cursor (project) | `<repo>/.cursor/skills/flutter-defect-triage/` |
| Claude Code | `~/.claude/skills/flutter-defect-triage/` |

`SKILL.md` must be at the root of that directory.

---

## Usage

Just describe the defect — no need to invoke a command first:

```
The channel member list Tab has a gray bar on the right
Hold-to-talk in Android WebView doesn't show the permission dialog
Is this a safe area issue?
Help me triage this one
```

The agent should first present a **pit alignment table**, not jump straight to editing files. You correct it ("this pit doesn't exist / that one is the real cause"), and only then does it proceed to investigation and fix.

After the fix lands, tell it to "backfill the Case": update `cases.md` + `cases-rag-index.md`.

---

## Two-Layer Architecture

| Layer | Location | Contents |
|-------|----------|----------|
| **Global skill** | `~/.cursor/skills/flutter-defect-triage/` | Workflow, templates, cross-repo Cases, business packs |
| **Project overlay** | `<repo>/.cursor/skills/flutter-defect-triage/` | Repo paths, requirement tables, dual-repo mappings, repo-specific Cases |

Keep global files short and business-agnostic. Product paths, protocols, and release slots go in the overlay. See the full example: [examples/project-overlay](examples/project-overlay/).

---

## Repository Structure

```
flutter-defect-triage/
├── SKILL.md                          # Workflow (agent entry point)
├── templates.md                      # Case / pack / alignment table templates
├── cases-rag-index.md                # Read the index first
├── cases.md                          # Fictional sample Cases 1–3
├── businesses/
│   ├── README.md                     # Business taxonomy
│   └── sample-club-web/              # Fictional business pack
│       ├── pack.md
│       └── hosts/android-webview-keyboard.md
├── examples/project-overlay/         # In-project stub template
├── setup
├── VERSION
└── LICENSE
```

The sample product **Harbor Club** is fictional, used to demonstrate three real-world pit categories: Tab shrinkage, data path divergence, and host permissions. Replace the slug and Cases when onboarding a real repository.

---

## Onboarding a Real Project

1. Run `bash setup` to install the global skill
2. Copy `examples/project-overlay/` → your target repo's `.cursor/skills/flutter-defect-triage/`
3. Create `businesses/<your-slug>/pack.md` and register it in `businesses/README.md`
4. Delete or replace the Harbor Club sample Cases
5. Run your first real defect through the full pipeline: align → fix → backfill

**Do not** commit internal repository URLs, App IDs, real protocols, ticket numbers, or employee paths to your public fork. This repository open-sources the **workflow**, not any company's business pack.

---

## Desensitization Note

This repository was rewritten after extracting the **methodology** from a production Cursor skill. The following have been removed:

- Real product names, campaign IDs, App IDs
- Internal Git / ticket / pipeline URLs
- Real protocol names, item IDs, payment channels
- Real Cases, commits, MRs, version allowlists
- Personal machine paths and company email addresses

Sample Cases retain only publicly shareable mechanisms: Flutter layout shrinkage, display path ≠ data path, and WebView permissions must go through the host bridge.

---

## Upgrading

```bash
git -C ~/.cursor/skills/flutter-defect-triage pull --ff-only
```

Or re-run `setup`.

---

## License

[MIT](LICENSE) © lnuxe