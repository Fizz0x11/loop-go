# loop-go

> Give a GitHub URL, AI runs the full loop: clone → test → introduce → wiki → review → handoff.

`loop-go` is an AI Agent skill that handles GitHub projects end-to-end. One trigger, one run, zero shortcuts.

**What it does**: Clone a project → run real tests → write a human-readable intro → save to your wiki → self-review → hand off to your future self.

**How to trigger**: Just say `loop-go <GitHub-URL>` to your AI Agent.

---

## Quick Start

```
loop-go https://github.com/owner/repo
```

Your AI Agent will automatically run the full 8-step loop.

---

## Architecture

```
loop-go (Skill #648)
└── loop-go-exec (Skill #648-1)
    ├── Step 1: Clone + Test (maker agent)
    ├── Step 2: Verify (checker agent)
    ├── Step 3: Write Wiki
    ├── Step 4: 631 Self-audit
    ├── Step 5: CE Review
    ├── Step 6: Self-reflection
    ├── Step 7: Human handoff
    └── Step 8: Ask about cron + Feishu card
```

### Dual-Agent Design

The loop enforces separation between:
- **Maker agent** — executes clone + tests, writes Wiki draft
- **Checker agent** — scores the output, rejects if < 8/10

> Same agent doing both = easy to fool yourself. The separation makes results real.

---

## The 8 Steps

| Step | Who | What |
|------|-----|------|
| 1 | Maker | Clone, read README, install deps, run a test command |
| 2 | Checker | Score 0-10: 8+ pass, 5-7 fix items, 0-4 retry |
| 3 | — | Write results to your Wiki (kdocs-cli for WPS, or any doc platform) |
| 4 | — | 631 self-audit: 5 components + external memory check |
| 5 | — | CE Review: Plan → Work → Review → Compound |
| 6 | — | Self-reflection: did this loop actually run? |
| 7 | — | Write human-readable handoff |
| 8 | — | Ask: want a cron job? Feishu card? |

---

## Screenshot

The presentation page (open `index.html` in any browser):

```
open index.html
```

Dark/light theme toggle, no dependencies, browser is the runtime.

---

## Prerequisites

- AI Agent with skill system (tested with Hermes Agent)
- `gh` CLI or `git` for cloning
- `kdocs-cli` for WPS Wiki (optional — replace with your wiki platform)

---

## File Structure

```
loop-go/
├── SKILL.md                         # Parent skill (trigger entry)
├── 648-1/
│   ├── SKILL.md                     # Execution layer
│   └── references/
│       └── connector/
│           ├── clone-and-test.md    # Maker agent instructions
│           └── verification.md       # Checker agent criteria
├── index.html                        # Presentation page (open in browser)
└── README.md
```

---

## Verification Scoring

| Score | Verdict | Action |
|-------|---------|--------|
| 8-10 | ✅ Pass | Continue to next step |
| 5-7 | ⚠️ Fix items | List what to fix, back to maker |
| 0-4 | 🔴 Fail | Retry from step 1 |

**Score dimensions**: A) Basic info (project type, README) + B) Reality check (did commands actually run?) + C) Wiki quality

---

## Self-Reflection Checklist (Step 6)

After each loop, the AI checks itself:

1. Did `clone` actually run?
2. Did test commands actually execute?
3. Was the handoff written?
4. Did CE Review get all three fields filled?
5. Was the self-check honest?
6. What's the one thing to fix next time?

---

## Customizing

**Wiki platform**: Replace `kdocs-cli` calls with your wiki API (Notion, Confluence, Obsidian, etc.)

**Workdir**: Set `$WORKDIR` to any directory you use for temporary project clones

**Skills framework**: Originally built for Hermes Agent — adapts to any AI Agent system that supports skill files + tool calls

---

## Why this exists

Most "AI reviews a repo" tools stop at: clone → read README → summarize. That's a demo, not a skill.

`loop-go` does the annoying parts:
- Actually runs the install commands
- Actually verifies the project works
- Actually writes the wiki so you don't have to
- Actually self-reviews so the loop improves itself

---

## License

MIT — use it, adapt it, make it yours.
