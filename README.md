# quiz-gate

An Agent Skill for Cursor and Claude Code that treats human understanding as a release criterion.

Before implementing a non-trivial change, the agent writes out the plan it would have written anyway — what it is going to do, how it works, why over the alternative — and then opens the quiz in that same reply. The skill does not impose a plan template; it only requires that the explanation is on screen before the questions are. The plan is how the developer learns the design; the quiz sits right below it, so reading and answering happen in one pass. You never have to ask for the quiz or reply `ready` — the plan and the questions arrive together. The agent stays read-only until the developer passes.

Both tools read the same `SKILL.md` format from the same directory layout, so one copy of `quiz-gate/` works in either.

## Install

Personal, available across all your projects:

```bash
# Cursor
mkdir -p ~/.cursor/skills && ln -s "$PWD/quiz-gate" ~/.cursor/skills/quiz-gate

# Claude Code
mkdir -p ~/.claude/skills && ln -s "$PWD/quiz-gate" ~/.claude/skills/quiz-gate
```

Project-scoped, shared with everyone using a repository:

```bash
# Cursor
mkdir -p /path/to/repo/.cursor/skills && cp -R quiz-gate /path/to/repo/.cursor/skills/

# Claude Code
mkdir -p /path/to/repo/.claude/skills && cp -R quiz-gate /path/to/repo/.claude/skills/
```

Keep the directory named `quiz-gate`: Claude Code derives the `/quiz-gate` command from the directory name and expects it to match the `name` field.

## Usage

The skill sets `disable-model-invocation: true`, which both tools honor, so it loads only when you name it:

```
/quiz-gate add idempotent webhook processing to the billing service
```

Remove that frontmatter line if you want the agent to apply the gate on its own whenever it detects feature or architecture work.

## Quiz size

| Change | Questions |
|---|---|
| Localized | 5 |
| Multi-component feature | 8 |
| Architectural, security-sensitive, or high-risk | up to 12 |

The agent asks the quiz through Cursor's `AskQuestion` or Claude Code's `AskUserQuestion` UI. Select an option for each question; do not type letter codes. Every answer must be correct to pass.

An incorrect answer blocks the gate and starts a teach-then-retest loop: the agent first explains the concept you missed and why your answer does not hold, then asks two or three new questions on that concept — different scenarios and different options, never a repeat of a question you have already seen. The loop continues until every missed concept is answered correctly.

## Portability note

`disable-model-invocation` is an extension that Cursor and Claude Code both support, but it is not part of the open Agent Skills specification, which allows only `name`, `description`, `license`, `compatibility`, `metadata`, and `allowed-tools`. Uploading this skill to claude.ai or the Skills API fails with an unexpected-key error until you remove that line, which also makes the gate model-invocable.

See [quiz-gate/SKILL.md](quiz-gate/SKILL.md) for the full instructions.
