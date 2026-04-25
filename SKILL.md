---
name: handoff
description: Write a complete session state capture to a HANDOFF file, then instruct user to /compact or /clear. Use before switching machines, hitting context limits, or handing off to a future session. Handles single-project, multi-project, and multi-machine sessions explicitly.
---

# /handoff — Session Handoff

> Capture everything a fresh Claude session needs to continue this work without losing the thread.

When invoked, perform the following steps in order. Do not ask for confirmation — just execute.

---

## Step 1: Gather Context

Collect the following (use Bash/Git/TaskList tools as needed):

1. **CWD** — `pwd` to know where the session is rooted
2. **Project markers** — does CWD or any ancestor contain `.git`, `package.json`, `CLAUDE.md`, or similar?
3. **Remote machines touched** — scan the conversation for `ssh <host>` invocations. List every host that was SSH'd into. This is critical for multi-machine sessions.
4. **Files actually edited** — the absolute paths of every file the session wrote to (local AND remote like `host:~/path/...`)
5. **Git state** — only for the LOCAL CWD if it's a git repo (`git status --short` + `git log --oneline -5`). Do NOT try to git-status remote paths.
6. **Active tasks** — check TaskList tool for in_progress or pending items
7. **Session audit** — review the full conversation and identify:
   - What was being built / worked on
   - What is complete vs. in-progress vs. blocked
   - Key decisions made, errors hit, approaches tried
   - Next concrete action needed

---

## Step 2: Classify the session and pick the write location

Use this decision tree EXPLICITLY. Do not guess. Print which path you took in the final output.

### Path A: Local single-project session
**Trigger**: CWD is inside a project (has `.git` / `package.json` / `CLAUDE.md` ancestor) AND no remote machines were touched.
**Write to**: `<project_root>/HANDOFF.md`
**Backup existing**: if a `HANDOFF.md` exists with different `# Handoff — <name>` header, rename it to `HANDOFF.md.bak.<YYYY-MM-DD-HHMMSS>` first.

### Path B: Generic CWD, single dominant project
**Trigger**: CWD is generic (`~`, `/tmp`, etc.) BUT 80%+ of file edits land in a single recognizable project directory.
**Write to**: that project's `HANDOFF.md`
**Backup existing**: same as Path A.

### Path C: Multi-machine / multi-project / orchestration session
**Trigger**: ANY of the following is true:
- 2+ remote machines were SSH'd into
- Files were edited across 2+ different project directories
- The session built or modified things that don't have a single "home" directory (e.g., dispatching agents, infrastructure work)

**Write to**: `~/.claude/handoffs/HANDOFF-<slug>-<YYYY-MM-DD>.md` (canonical archive) AND `~/HANDOFF.md` (latest pointer copy).
- `<slug>` is a kebab-case 2-4 word summary of the session.
- `~/.claude/handoffs/` — create with `mkdir -p` if it doesn't exist.
- Both files have IDENTICAL content. The archive copy is for history; the `~/HANDOFF.md` copy is the "latest" so the resume prompt has a stable path.
- Before overwriting `~/HANDOFF.md`, if its existing `# Handoff — <name>` header doesn't match the current session, MOVE it to `~/.claude/handoffs/HANDOFF-<old-slug>-<old-date>.md.archive` (preserve, don't delete).

### Path D: Last resort
**Trigger**: None of the above match cleanly.
**Write to**: `~/HANDOFF.md` and announce the choice clearly in the output.

---

## Step 3: Write the HANDOFF file

Use this exact structure (adapt content, keep structure):

```markdown
# Handoff — [Session Topic] — [YYYY-MM-DD]

> Read this at the start of the next session. Paste the Resume Prompt below as your first message.

## Session Type
- **Path used**: [A: Local single-project | B: Generic CWD single project | C: Multi-machine | D: Last resort]
- **CWD when written**: [absolute path]
- **Machines touched**: [list every SSH host, or "local only"]

## Scope
- **Project(s)**: [name + path. For multi-machine, list each: `host1:~/tools/foo/`, `host2:~/tools/bar/`, etc.]
- **Public URLs touched**: [if any are running services]
- **Local git** (only for local CWD if it's a repo): [branch, last commit, or "no git" or "remote-only session"]

## What We Were Doing

[2-4 sentences describing the task, goal, and current approach. Be specific enough that a fresh Claude
session can understand without reading the full history.]

## State

### Done
- [Completed item — be specific]
- [Completed item]

### In Progress
- [Current active work — where exactly it was left]

### Blocked / Pending
- [If any blockers or things waiting on external action]

## Next Steps

1. [Most immediate next action — specific file, function, command. Use absolute paths with machine prefix if remote: `host:~/path/to/file`]
2. [Second step]
3. [Third step if applicable]

## Key Files

| File | Role |
|------|------|
| `[absolute path with machine prefix if remote]` | [what it does or what changed] |
| `[absolute path]` | [what it does or what changed] |

## Context to Know

[Bullet list of non-obvious facts, gotchas, decisions, or environment notes a fresh session would
need. Skip anything obvious from the code itself.]

- [Context item]
- [Context item]

## Resume Prompt

Paste this as your first message in the new session:

---
Continuing work on [session topic].

Read HANDOFF.md at [ABSOLUTE PATH — never assume CWD] for full context.

Current task: [one-line description]
Next action: [specific first thing to do, with absolute paths]
---
```

---

## Step 4: Confirm and Hand Off

After writing the file(s):

1. Print which classification path was used (A/B/C/D) and why
2. Print the FULL ABSOLUTE PATH(S) of every file written, on their own lines, copy-paste friendly
3. Briefly summarize what was captured (3-5 bullets)
4. Tell the user:

> **HANDOFF written.**
> Primary: `<absolute path>`
> [Archive: `<archive path>` (only for Path C)]
>
> Now run `/compact` to compress context (keeps conversation, cheaper), or `/clear` to start completely fresh.
>
> In the new session, run `claude` from any directory and say:
> *"Read HANDOFF.md at <absolute primary path>"*
>
> The path is absolute — it works regardless of where the new session starts.

---

## Notes

- **Keep the body under 100 lines** — dense and actionable, not comprehensive. Future Claude can read code.
- **The Resume Prompt is the most important part** — it must be copy-pasteable and complete enough to orient a fresh session in under 10 seconds. ALWAYS use the absolute path of the HANDOFF file.
- **Never assume the next session opens in a specific CWD** — the user might start it from a different machine, a different terminal, a different directory. All paths in the handoff must be absolute.
- **For remote files, use the `<host>:<path>` notation** (e.g., `mybox:~/tools/foo/.env`) so the next session immediately knows it's not local.
- **Do not include this session's full conversation** — distill it, don't transcribe it.
- **If multiple projects were touched** (Path C), they all live in ONE handoff document with sections per machine/project. Don't write multiple handoff files for one session — that fragments the context.
- **Backup, don't overwrite blindly**: if an existing HANDOFF.md is for a different project/session, rename it with a timestamp suffix before writing the new one.
