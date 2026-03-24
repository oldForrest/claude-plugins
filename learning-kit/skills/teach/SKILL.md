---
name: teach
description: Run an interactive teaching session from a /study learning plan. Picks up where the user left off, walks through codebase topics with real code examples and traced flows, and tracks progress. Use when the user says /teach, "teach me", "next topic", "continue studying", "walk me through the next one", "keep going", or any variation of wanting to learn more about a repo they've already run /study on. Also triggers on "explain [topic]" or "show me how [thing] works" when a learning plan exists.
user_invocable: true
invocation: /teach
---

# Teach — Interactive Codebase Teaching Sessions

Walk the user through topics from a `/study` learning plan. Each session covers one topic in depth with real code, traced flows, and practical understanding.

## Step 1: Load State

Read `.claude/learning/plan.json` and `.claude/learning/progress.json` from the current repo.

**If no plan exists:** Tell the user to run `/study` first. Don't improvise a teaching session without a plan — the plan gives structure and tracks progress.

**If a plan exists:** Determine where they are:
- What topics are completed vs not started?
- What was the last session about?
- What's the natural next topic based on prerequisites?

## Step 2: Pick a Topic

Present the user with their current state and a recommendation:

```
You've completed 3/10 topics. Last session covered "Extraction Pipeline".

Next up: "Hybrid Search (Semantic + BM25 + RRF)" — complexity: high
  This builds on the DI container you already know and is one of the most
  interesting parts of the codebase.

Ready to dive in, or want to pick a different topic?
```

If the user names a specific topic or area, match it to the plan. If they haven't met the prerequisites, mention it but don't block them — they're an adult.

If they just say "teach me" or "next", go with your recommendation.

## Step 3: Teach the Topic

This is the core of the skill. A good teaching session has five parts:

### 3a. Set Context (1-2 minutes of reading)

Briefly explain what this topic covers, why it matters, and how it fits into the bigger picture. Connect it to topics they've already learned.

### 3b. Walk Through Key Files

Read the important files and explain what's happening. Focus on:

- **Why** things are structured this way, not just what the code does
- **Design decisions** — why this approach over alternatives
- **How pieces connect** — trace imports, function calls, data flow between modules

Don't dump entire files. Read the relevant sections and narrate them. Pull in specific line ranges that illustrate the point.

### 3c. Trace a Concrete Flow

Pick one real operation and trace it end-to-end through the code. This is the most valuable part — it shows how abstractions connect in practice.

Read the actual code at each step. Don't hand-wave.

### 3d. Highlight Patterns & Gotchas

Point out things that aren't obvious from reading code linearly:

- Clever patterns worth learning from
- Non-obvious side effects or ordering dependencies
- Places where the code does something surprising
- Common mistakes a new contributor might make
- Performance implications or scaling considerations

### 3e. Summarize & Connect Forward

Wrap up with 3-5 key takeaways and connect to what comes next.

## Step 4: Check Understanding

After the summary, ask 2-3 quick comprehension questions. These aren't quiz-level — just quick checks that the key points landed.

If they get something wrong, clarify it right away. This is teaching, not testing.

## Step 5: Update Progress

After the session, update both files:

**plan.json** — Set the topic's status to `"completed"`.

**progress.json** — Add a session entry:

```json
{
  "date": "YYYY-MM-DD",
  "topic_id": "t5",
  "topic_title": "Topic Title",
  "notes": "Brief summary of what was covered and any areas where understanding seemed shaky"
}
```

The notes should be honest — capture what was covered and any areas where understanding seemed shaky. This feeds into `/quiz` later.

## Step 6: What's Next

End with a pointer forward:

- Suggest the next topic based on prerequisites and completion
- Mention `/quiz` as an option to test what they've learned
- If they've completed all topics, congratulate them and suggest a full review quiz

## Pacing

- Don't rush. If a topic is complex (complexity 3), it's OK for the session to be longer.
- Don't over-explain things the user clearly already knows. Calibrate to their level.
- If the user asks a question mid-session, answer it fully. Tangents that deepen understanding are good.
- If a topic turns out to be simpler than expected, combine it with a related one in the same session. Update both topics' status.

## Tone

- Knowledgeable colleague walking you through their codebase, not a lecturer
- "Here's the interesting part" energy — share genuine enthusiasm for clever solutions
- Direct about weaknesses too — "this part is a bit rough" or "this could be simpler"
- Reference concrete code, not abstractions. Line numbers, function names, actual values
