---
name: quiz
description: Interactive quiz that tests the user's understanding of a codebase based on a /study learning plan. Generates questions of varying difficulty and style (multiple choice, open-ended, code-reading), adapts to performance, and tracks progress over time. Use when the user says /quiz, "test me", "quiz me on this repo", "how well do I know this", or any variation of wanting to be tested on their codebase knowledge. Also trigger on casual phrases like "let's see if I learned anything" or "check my understanding".
user_invocable: true
invocation: /quiz
---

# Quiz — Codebase Knowledge Testing

Interactive quiz system that tests the user's understanding of a repo based on a learning plan created by `/study`. Adapts difficulty based on performance and tracks mastery over time.

## Step 1: Load the Learning Plan

Read `.claude/learning/plan.json` and `.claude/learning/progress.json` from the current repo.

**If no plan exists:** Tell the user to run `/study` first. Offer to do a quick ad-hoc quiz instead — you'll explore the repo briefly and generate 5 general questions without a structured plan.

**If a plan exists:** Analyze what topics have been studied vs not, and any previous quiz scores.

## Step 2: Select Quiz Scope

Ask the user what they want to be quizzed on:

- **Full review** — All completed topics, weighted toward ones they haven't been quizzed on yet
- **Specific topic** — Let them pick from the list
- **Weak spots** — Focus on topics where previous quiz scores were low
- **Quick 5** — Five random questions across everything

Default to "Full review" if they just say "quiz me".

## Step 3: Generate Questions

Generate 5-10 questions (depending on scope) using a mix of these styles:

### Question Types

**Multiple Choice** (good for pattern recognition, conventions)
```
What design pattern does `src/middleware/auth.ts` use for token validation?

a) Strategy pattern — swappable validators
b) Chain of responsibility — middleware pipeline
c) Observer — event-based validation
d) Decorator — wrapping the base handler
```

**Code Reading** (good for understanding data flow)
```
Look at this function signature from the codebase:

  async function resolveOrder(orderId: string, ctx: RequestContext): Promise<OrderResult>

Without looking at the implementation — based on what you've learned:
1. Where does this function get called from?
2. What database tables does it likely touch?
3. What could cause it to throw?
```

**Open-Ended** (good for architecture understanding)
```
Explain how a request flows from the API gateway to the database
in this application. What are the key middleware steps?
```

**Gotcha Questions** (good for tribal knowledge)
```
A new developer adds a migration that renames the `users.email` column.
What would break, based on how this codebase is structured?
```

**"What Would You Change"** (good for deep understanding)
```
If you needed to add WebSocket support to this app's notification system,
which files would you need to modify and why?
```

### Question Generation Rules

- Questions must be answerable from what the learning plan covers — don't test obscure implementation details
- Reference real files, functions, and patterns from the actual codebase
- For multiple choice: all options should be plausible, not just one obvious answer and three jokes
- Scale difficulty based on topic complexity and user's previous performance
- Mix question types — don't do 10 multiple choice in a row

## Step 4: Run the Quiz

Present questions one at a time. For each question:

1. Show the question clearly
2. Wait for the user's answer
3. Grade it:
   - **Correct** — Confirm and optionally add a bonus insight they might not have known
   - **Partially correct** — Acknowledge what they got right, explain what they missed
   - **Incorrect** — Explain the correct answer with a reference to the relevant code (`file:line`). Be encouraging, not condescending
4. Track the score

For code-reading and open-ended questions, evaluate the substance of their answer rather than expecting exact wording. The user might express the right concept differently than you would.

### Adaptive Difficulty

- If they get 3+ in a row correct: increase difficulty (ask about interactions between systems, edge cases, non-obvious implications)
- If they get 2+ in a row wrong: dial back (focus on fundamentals, offer hints, use multiple choice instead of open-ended)
- Always tell them when you're adjusting: "You're crushing it — let me make these harder" or "Let's revisit the basics on this one"

## Step 5: Score and Record

After all questions, show a summary:

```
Quiz Results — [Topic or "Full Review"]
Score: 7/10 (70%)

Strong areas:
  - Project structure and entry points
  - Authentication middleware flow

Needs review:
  - Database migration patterns (missed Q4, Q8)
  - Error handling conventions (partial on Q6)

Suggested next steps:
  - Re-study "Data Layer" topic focusing on migration workflow
  - Try /quiz again on weak spots after reviewing
```

Save the results to `.claude/learning/progress.json`:

```json
{
  "quiz_scores": [
    {
      "date": "YYYY-MM-DD",
      "scope": "full_review",
      "topics_tested": ["t1", "t2", "t3"],
      "score": 7,
      "total": 10,
      "per_topic": {
        "t1": {"correct": 3, "total": 3},
        "t2": {"correct": 2, "total": 4},
        "t3": {"correct": 2, "total": 3}
      },
      "weak_areas": ["database migrations", "error handling conventions"]
    }
  ]
}
```

Update `mastered_topics` (consistently >80%) and `struggling_topics` (<60%) based on cumulative quiz history.

## Ad-Hoc Quiz (No Learning Plan)

If there's no `.claude/learning/plan.json`, do a quick codebase scan:

1. Read the project structure, entry points, and key config files
2. Generate 5 general questions covering: project purpose, structure, main technologies, data flow, and one gotcha
3. Don't save progress (no plan to track against) — just run the quiz and show the score
4. Suggest running `/study` afterward if they want structured learning

## Tone

- Keep it engaging, not clinical — this should feel like a conversation, not a standardized test
- Celebrate correct answers genuinely but briefly
- On wrong answers, teach rather than just correct — the quiz is a learning tool
- If the user is clearly guessing, offer a hint before revealing the answer
- Use humor sparingly and naturally — "trick question" energy is fine, smugness is not
