---
name: study
description: Explore a codebase and build a structured learning plan to teach the user how the repo works. Creates a progressive curriculum covering architecture, key abstractions, data flow, patterns, and gotchas. Use when the user says /study, "teach me this repo", "help me understand this codebase", "create a learning plan", "I want to learn how this works", or any variation of wanting to understand a codebase they're working in. Even casual phrases like "walk me through this repo" or "what should I know about this project" should trigger this skill.
user_invocable: true
invocation: /study
---

# Study — Repo Learning Plan Generator

Build a structured, progressive learning plan that teaches the user how a codebase works. Calibrate explanations to the user's experience level — if you know their background, lean into it; otherwise, assume a competent developer who prefers understanding architecture and patterns over hand-holding.

## Step 1: Explore the Repo

Quickly survey the codebase to understand its shape. Run these in parallel where possible:

1. **Project identity** — Read `package.json`, `README.md`, `CLAUDE.md`, `pyproject.toml`, `Cargo.toml`, or equivalent. What is this project? What framework/language?
2. **Directory structure** — `ls` the top-level dirs and key subdirectories. Map out the module boundaries.
3. **Entry points** — Find the main entry point(s): `main`, `index`, server startup, CLI entry, etc.
4. **Config & environment** — Scan for config files, env templates, docker-compose, CI configs. What services does this depend on?
5. **Data layer** — Look for schemas, migrations, models, API contracts (OpenAPI, GraphQL schema, Prisma schema, etc.)
6. **Key abstractions** — Grep for classes, major exports, middleware, hooks, or patterns that form the backbone.

Don't read every file — skim strategically. The goal is a map, not a novel.

## Step 2: Identify Learning Topics

Based on the exploration, identify 5-10 learning topics organized from foundational to advanced. Good topic categories:

- **Foundation**: Project structure, build system, how to run it, dev workflow
- **Core architecture**: How the pieces fit together, request/data flow, key abstractions
- **Domain logic**: The business logic — what this thing actually does and why
- **Data & state**: Database schema, state management, caching, data flow
- **Patterns & conventions**: Code style, error handling, testing patterns, naming conventions
- **Integration points**: External APIs, services, message queues, auth systems
- **Gotchas & tribal knowledge**: Non-obvious things that would bite a new developer

Each topic should have:
- A clear title
- 2-3 sentence description of what it covers
- Key files to read (with paths)
- Prerequisites (which topics should come first)
- Estimated complexity (1-3, where 1 is straightforward and 3 requires careful study)

## Step 3: Create the Learning Plan

Create the `.claude/learning/` directory if it doesn't exist.

Save the plan to `.claude/learning/plan.json`:

```json
{
  "repo": "repo-name",
  "created": "YYYY-MM-DD",
  "summary": "One paragraph describing what this repo is and what makes it interesting/complex",
  "topics": [
    {
      "id": "t1",
      "title": "Project Structure & Dev Workflow",
      "description": "How the repo is organized, how to run it, build system",
      "key_files": ["package.json", "src/index.ts", "docker-compose.yml"],
      "prerequisites": [],
      "complexity": 1,
      "status": "not_started",
      "subtopics": [
        "Directory layout and module boundaries",
        "Build and run commands",
        "Environment setup"
      ]
    }
  ]
}
```

Initialize `.claude/learning/progress.json`:

```json
{
  "repo": "repo-name",
  "sessions": [],
  "quiz_scores": [],
  "mastered_topics": [],
  "struggling_topics": []
}
```

Also create `.claude/learning-kit.json` if it doesn't already exist — this enables the onboarding hook:

```json
{
  "mode": "nudge",
  "min_topics_completed": 3,
  "min_quiz_score": null,
  "required_topics": [],
  "message": null
}
```

Mention to the user that this config was created and explain what it does: the learning-kit plugin will gently remind developers who haven't completed enough topics. They can change `"mode"` to `"gate"` for a harder requirement or `"off"` to disable.

## Step 4: Present the Plan

Show the user a clean summary:

1. **Repo overview** — 2-3 sentences on what this project is
2. **Learning roadmap** — Numbered list of topics in recommended order, with complexity indicators
3. **Suggested starting point** — Which topic to tackle first and why
4. **Time estimate** — Rough sense of how many study sessions the full plan would take

Ask if they want to:
- **Start studying now** — Begin with the first topic (or one they pick)
- **Adjust the plan** — Reorder, add, or remove topics
- **Focus on a specific area** — Skip to what they care about most

## Step 5: Teach a Topic (if they want to start)

When teaching a topic:

1. **Set context** — Briefly explain what this topic covers and why it matters
2. **Walk through key files** — Read the important files and explain what's happening, focusing on the *why* not just the *what*
3. **Trace a flow** — Pick a concrete example (a request, a command, a data transformation) and trace it through the relevant code
4. **Highlight patterns** — Point out conventions, clever solutions, or non-obvious design decisions
5. **Flag gotchas** — Anything surprising or easy to misunderstand
6. **Summarize** — 3-5 key takeaways for this topic

After teaching, update the topic's status in `plan.json` to `"completed"` and log the session in `progress.json`:

```json
{
  "date": "YYYY-MM-DD",
  "topic_id": "t1",
  "topic_title": "Project Structure & Dev Workflow",
  "notes": "Brief summary of what was covered"
}
```

## Resuming a Previous Plan

If `.claude/learning/plan.json` already exists, don't start over. Read it and `progress.json`, then:

1. Show what's been covered so far
2. Suggest the next topic based on prerequisites and what's completed
3. Ask if they want to continue, review a previous topic, or adjust the plan

## Tone

- Talk to the user like a knowledgeable colleague, not a textbook
- Use concrete code references, not abstract descriptions
- When something is genuinely complex, say so — don't oversimplify
- If a design decision seems questionable, mention it neutrally ("this is an unusual choice — here's what it accomplishes")
- Keep explanations tight. The user can ask for more detail on anything
