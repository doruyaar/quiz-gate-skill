---
name: quiz-gate
description: Require the developer to demonstrate understanding of a proposed architecture or non-trivial code change before the agent implements it. Use for feature development, architectural changes, unfamiliar codebases, and implementation plans where the human must retain ownership. Skip trivial edits and requests limited to explanation, research, or review.
disable-model-invocation: true
---

# Quiz Gate

Treat human understanding as a release criterion.

The agent may inspect files, investigate the codebase, ask clarifying questions, and design a solution before the gate.

The agent must not edit files, run mutating commands, or begin implementation until the developer passes the Quiz Gate.

## 0. Scope check

Run the gate for feature development, architectural changes, and other non-trivial modifications.

Skip the gate for:

- trivial or mechanical edits;
- requests limited to explanation, research, or review;
- work covered by a gate the developer already passed in this session, unless the plan has materially changed.

## 1. Establish the proposed solution

Inspect the relevant code and requirements.

Present a concise implementation plan covering:

- intended behavior and scope;
- affected components and their responsibilities;
- control flow and data flow;
- important design choices;
- significant alternatives that were rejected and why;
- failure modes and recovery behavior;
- security, concurrency, privacy, or data-integrity risks;
- testing and observability;
- deployment, migration, or rollback consequences when relevant.

Resolve material ambiguities before opening the gate.

The quiz must test the agreed solution. Do not test unstated assumptions, arbitrary implementation details, or decisions that have not been explained to the developer.

## 2. Open the Quiz Gate

Create an adaptive multiple-choice quiz:

- Use 5 questions for a localized change.
- Use 8 questions for a multi-component feature.
- Use up to 12 questions for an architectural, security-sensitive, or high-risk change.
- Give four options per question.
- Include exactly one best answer.
- Randomize the position of correct answers.
- Do not reveal the answers before submission.

Ask all questions in one numbered batch.

Request answers in a compact format such as:

`1B 2D 3A 4C 5B`

Tell the developer how many questions there are, but do not reveal the distribution of correct answers.

Fix the answer key before presenting the quiz. Do not re-derive the intended answers after reading the developer's submission.

## 3. Test ownership, not memory

Test whether the developer can reason about, modify, debug, and operate the proposed solution.

Cover the relevant categories:

1. The feature's purpose and primary invariant.
2. Component responsibilities and boundaries.
3. Control flow or data flow.
4. Design rationale and tradeoffs.
5. Failure behavior and recovery.
6. Security, privacy, concurrency, or data integrity.
7. Testing and observability.
8. The change surface of a likely future requirement.

Include at least one scenario-based question that asks the developer to predict what happens under a specific condition.

Prefer questions such as:

- What happens if this dependency fails after the state change?
- Which component is responsible for preventing duplicate processing?
- Why was this design chosen instead of the main alternative?
- Where should investigation begin if this fails in production?
- Which components must change if a particular requirement is added?
- Which invariant would be violated by this proposed modification?

Avoid trivia such as:

- filenames;
- exact class or function names;
- syntax;
- configuration values with no architectural significance;
- facts visible in a single line;
- wording copied directly from the plan.

Do not ask about parts of the system that the proposed change does not affect.

## 4. Grade the gate

Grade answers against the agreed plan and repository evidence, not merely the agent's preferred design.

If the submission is incomplete or malformed, ask the developer to resubmit the missing items before grading. Do not infer unanswered questions.

### Passing

If every answer is correct:

1. State `Quiz Gate: PASS`.
2. Briefly summarize the areas the developer demonstrated understanding of.
3. Proceed with implementation.

### Blocking

If any answer is incorrect:

1. State `Quiz Gate: BLOCKED`.
2. Identify the misunderstood concept.
3. Explain that concept briefly.
4. Ask a new multiple-choice question testing the same concept through a different scenario.
5. Do not repeat the original wording.

Continue targeted remediation until every tested concept has been answered correctly at least once.

Do not retest concepts the developer has already passed unless later answers reveal a contradictory understanding.

If a question was ambiguous, withdraw it and replace it. Do not penalize the developer.

If the plan changes during the discussion, update the plan and replace only the questions affected by that change.

## 5. Detect guessing when possible

Multiple-choice answers can be guessed.

If a selected answer conflicts with the developer's accompanying explanation or another answer:

1. Do not pass that concept yet.
2. Point out the apparent contradiction.
3. Ask one short scenario question testing the same concept.
4. Pass the concept only after the contradiction is resolved.

Do not require written explanations for every answer by default. Quiz Gate should add useful friction, not turn every change into an interview.

## 6. Preserve the gate

While the gate is blocked, the agent may:

- explain the proposed solution;
- answer questions;
- inspect the repository;
- revise the plan;
- perform other read-only investigation.

While blocked, the agent must not:

- edit or create project files;
- install dependencies;
- change configuration;
- run migrations;
- commit code;
- execute deployment actions;
- begin implementing the requested change.

Do not silently waive Quiz Gate because the task is urgent or because the implementation appears straightforward after planning.

### Explicit waiver

Waive the gate only when the developer directly asks to skip it. State `Quiz Gate: WAIVED`, note that human understanding was not verified, and proceed. Never infer a waiver from urgency, impatience, or silence.

## 7. Implement the agreed solution

After the developer passes, implement only the agreed plan.

Continue using normal engineering practices, including code review, tests, static analysis, and verification. Passing Quiz Gate confirms human understanding; it does not prove that the implementation is correct.

If implementation reveals a material architectural change:

1. Pause before making that change.
2. Explain why the original plan must change.
3. Present the proposed delta.
4. Open a small Delta Quiz Gate containing 1–3 questions about the new decision.
5. Continue only after the delta is passed.

Do not reopen the gate for minor implementation details that do not change the developer's mental model of the system.

## 8. Finish with an ownership summary

After implementation and verification, provide a short summary containing:

- what changed;
- where the main responsibilities now live;
- the most important invariant;
- the primary failure or operational risk;
- where the developer should begin investigating if the feature fails.

Do not quiz the developer again unless the final implementation materially differs from the plan they passed.
