# learning-kit

Codebase onboarding toolkit for Claude Code. Three skills that help developers learn any repo through structured study, interactive teaching, and adaptive quizzing.

## Skills

- **`/study`** — Explore a repo and build a progressive learning plan (5-10 topics, foundational to advanced)
- **`/teach`** — Interactive teaching sessions that walk through topics with real code, traced flows, and comprehension checks
- **`/quiz`** — Adaptive quizzes with multiple choice, code-reading, open-ended, and gotcha questions. Tracks mastery over time.

## Onboarding Hook

A `SessionStart` hook checks developer progress and nudges (or gates) them until they've completed enough onboarding. Repos opt in by adding a `.claude/learning-kit.json` config file (auto-created by `/study`).

### Configuration

Add `.claude/learning-kit.json` to your repo:

```json
{
  "mode": "nudge",
  "min_topics_completed": 3,
  "min_quiz_score": null,
  "required_topics": [],
  "message": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `mode` | `"nudge"` \| `"gate"` \| `"off"` | `nudge` prints a reminder. `gate` blocks until requirements are met. `off` disables the hook. |
| `min_topics_completed` | number | Minimum topics a developer must complete before the hook goes silent |
| `min_quiz_score` | number \| null | Optional minimum quiz score (0-100) to require |
| `required_topics` | string[] | Optional list of specific topic IDs that must be completed |
| `message` | string \| null | Optional custom message to show instead of the default |

## Install

```bash
claude plugins install oldForrest/learning-kit
```

## How it works

1. A team lead runs `/study` in the repo to create a learning plan
2. The plan and config get committed to the repo
3. When a new developer opens the repo in Claude Code, the hook reminds them to onboard
4. They run `/teach` to work through topics and `/quiz` to test themselves
5. Once they've met the thresholds, the hook goes silent

## Data files

All learning data lives in `.claude/learning/` within each repo:

- `plan.json` — The learning curriculum (topics, prerequisites, status)
- `progress.json` — Session history, quiz scores, mastery tracking

These can be committed to the repo (shared plan) or gitignored (individual progress). A typical setup commits `plan.json` and `learning-kit.json` but gitignores `progress.json`.
