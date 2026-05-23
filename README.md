# ormus-handoff

> Session state capture for Claude Code. Resume work tomorrow exactly where you left off — even on a different machine.

A Claude Code skill that writes a complete handoff document at session end so a fresh session can pick up the thread without re-explaining context.

## Why

Long Claude sessions hit context limits, get compacted, or just end. Resuming the next day usually means re-explaining everything — what you were building, what's done, what's blocked, what files matter. That's wasted tokens and wasted thought.

`/handoff` distills the entire session into one structured `HANDOFF.md` file with a copy-pasteable resume prompt. The next session reads it and continues immediately.

It also handles the harder cases: multi-machine sessions where you SSH'd into 3 boxes, multi-project sessions where edits landed in different repos, and orchestration sessions that don't have a single "home" directory.

## Install

Drop the skill into your Claude Code skills directory:

```bash
git clone https://github.com/HermeticOrmus/ormus-handoff ~/.claude/skills/handoff
```

Or as a Claude Code plugin:

```bash
claude plugin marketplace add HermeticOrmus/ormus-handoff
```

Restart Claude Code so the skill registry picks it up.

## Usage

In any Claude Code session, invoke:

```
/handoff
```

The skill executes in 4 steps:

1. **Gathers context** — CWD, git state, edited files, machines touched, active tasks, conversation summary
2. **Classifies the session** — single-project (Path A), generic CWD with one dominant project (B), multi-machine/multi-project (C), or last-resort (D)
3. **Writes HANDOFF.md** at the right location for the session type
4. **Tells you what to do next** — `/compact` or `/clear`, then paste the Resume Prompt in the new session

## Example output

```
HANDOFF written. (Path C — multi-machine)
Primary: /home/you/HANDOFF.md
Archive: /home/you/.claude/handoffs/HANDOFF-deploy-fix-2026-04-25.md

Captured:
  • 3 machines touched (local, prod-1, prod-2)
  • 7 files edited across 2 repos
  • 1 task in_progress, 2 done
  • Next action: rerun migration on prod-2 after the schema patch lands
```

## How it works

The skill is pure documentation — no scripts, no dependencies. It instructs Claude to:

- Use Bash/Git tools to gather session state
- Apply a strict classification decision tree (4 paths)
- Write a `HANDOFF.md` with a fixed structure: Session Type → Scope → What We Were Doing → State (Done/In Progress/Blocked) → Next Steps → Key Files → Context to Know → **Resume Prompt** (the copy-pasteable one-liner that bootstraps the next session)

The Resume Prompt always uses absolute paths so it works no matter which terminal, machine, or directory the next session opens from.

## Pairs with

- **[ormus-pickup](https://github.com/HermeticOrmus/ormus-pickup)** — the entry ritual. Reads a HANDOFF and restores context. Inverse of /handoff.
- **[ormus-absorb](https://github.com/HermeticOrmus/ormus-absorb)** — distill conversation knowledge into persistent memory. /handoff captures *task state*; /absorb captures *understanding*.
- **[ormus-explore](https://github.com/HermeticOrmus/ormus-explore)** — token-efficient AST-based code search.
- **[ormus-vibe-proof](https://github.com/HermeticOrmus/ormus-vibe-proof)** — security hardening for vibe-coded full-stack apps.
- **[ormus-meta-prompting](https://github.com/HermeticOrmus/ormus-meta-prompting)** — categorical foundations for AI prompt engineering.

Together they form the **ormus session lifecycle** — composable Claude Code skills for doing serious work across days, machines, and context resets.

## License

MIT. See [LICENSE](LICENSE).

## Origin

Distilled from real multi-day, multi-machine Claude Code sessions. Every classification path (A/B/C/D) maps to a real failure mode that lost work before this skill existed.

---

## Part of the Libre Open-Source Stack for Claude Code

This repository is part of a growing family of open-source toolkits for Claude Code, each focused on a specific lane:

- [LibreUIUX-Claude-Code](https://github.com/HermeticOrmus/LibreUIUX-Claude-Code) — UI/UX system (152 agents, 70 plugins, 76 commands, 74 skills)
- [LibreGEO-Claude-Code](https://github.com/HermeticOrmus/LibreGEO-Claude-Code) — AI-search optimization for ChatGPT, Perplexity, Gemini, Google AI Overviews
- [LibreEmbed-Claude-Code](https://github.com/HermeticOrmus/LibreEmbed-Claude-Code) — Embedded systems, firmware, and IoT development
- [LibreGameDev-Claude-Code](https://github.com/HermeticOrmus/LibreGameDev-Claude-Code) — Game development across Godot, Unity, Unreal
- [LibreFinTech-Claude-Code](https://github.com/HermeticOrmus/LibreFinTech-Claude-Code) — Financial technology development

Star the family, not just one — that's how the suite stays coherent.
