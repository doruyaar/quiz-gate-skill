---
name: quiz-gate
description: Require the developer to demonstrate understanding of a proposed architecture or non-trivial code change before the agent implements it. Use for feature development, architectural changes, unfamiliar codebases, and implementation plans where the human must retain ownership. Skip trivial edits and requests limited to explanation, research, or review.
disable-model-invocation: true
---

# Quiz Gate

Treat human understanding as a release criterion.

The agent may inspect files, investigate the codebase, ask clarifying questions, and design a solution before the gate.

The developer learns the solution from the plan, not from the quiz. Write the complete plan as visible text, then open the quiz in that same reply.

**The plan and the quiz are one turn.** Writing the plan and ending the turn is a failed gate, even if the plan is excellent. The developer should never have to ask for the quiz, say `ready`, or send any message between reading the plan and answering the first question. If you are about to end a turn whose last content is plan text, you are not done: call the question tool now.

The agent must not edit files, run mutating commands, or begin implementation until the developer passes the Quiz Gate.

## 0. Scope check

Run the gate for feature development, architectural changes, and other non-trivial modifications.

Skip the gate for:

- trivial or mechanical edits;
- requests limited to explanation, research, or review;
- work covered by a gate the developer already passed in this session, unless the plan has materially changed.

## 1. Teach the proposed solution

Inspect the relevant code and requirements, decide the solution, then explain that decision to the developer. This is the teaching step.

Explain it the way you would explain it anyway. The plan the developer reads here is the same plan you would write for this change without the gate: what you are going to do, how it works, why you chose it over the alternative you weighed. Quiz Gate does not ask for a different kind of document, and it does not add sections. It only requires that the explanation exists as visible text before the quiz opens.

Let the change decide the depth. A localized change is a few paragraphs. An architectural one needs the reasoning behind the structure, how it behaves when something fails, and what it costs to operate. Say what is actually load-bearing for this change and leave out what is not; a plan padded to satisfy a checklist teaches less than a short one that names the real decisions.

The plan is how the developer gains the knowledge the quiz will require. Do not treat the quiz as the explanation.

**Write the plan out before calling the question tool, and call the question tool before ending the turn.** Both halves matter. Emit the entire plan as visible chat text, then open the quiz on the way out of the same turn. The developer reads down the plan and answers immediately, with no round trip in between.

A status line such as "here is the full plan" is not the plan. If the question tool is called before the plan text is written, the developer sees only the question widget and the explanation is lost.

The last thing that happens in this turn is the question tool call. Not a summary, not a closing line, and above all not a handoff. Do not write anything resembling "let me know when you have read it", "ready for the quiz?", "say ok and I will start", or "next I will quiz you" — every one of those turns the gate into an extra round trip the developer has to initiate, which is exactly what this step exists to avoid. Announcing the quiz is not opening it.

Long plans do not change this. Finishing a substantial explanation feels like a natural stopping point; it is not one here. The turn ends when the questions are on screen.

Resolve material ambiguities before opening the gate. Once the plan is written, there is nothing left to wait for.

The one thing the plan must satisfy: a developer who reads it, and nothing else, can answer every question in the quiz and then own the change. If a question would test something the plan never said, either say it in the plan or drop the question. Do not test unstated assumptions, arbitrary implementation details, or decisions that have not been explained to the developer.

## 2. Open the Quiz Gate

Open the quiz in the same turn as the plan, directly after the plan text, without waiting for the developer to respond or ask for it.

Create an adaptive multiple-choice quiz:

- Use 5 questions for a localized change.
- Use 8 questions for a multi-component feature.
- Use up to 12 questions for an architectural, security-sensitive, or high-risk change.
- Give four options per question.
- Include exactly one best answer.
- Randomize the position of correct answers.
- Do not reveal the answers before submission.

Present every question through the host's structured question UI so the developer selects options instead of typing letter codes:

- Cursor: `AskQuestion`
- Claude Code: `AskUserQuestion`

Do not ask for answers in forms such as `1B 2D 3A`. Do not list A/B/C/D as the choices the developer is meant to type.

Map the quiz onto the tool:

- One question object per quiz item, single-select.
- Four options whose visible text is the choice itself, not a letter.
- Do not add an Other option; the host may append one.
- Do not mark, recommend, or otherwise distinguish the correct option.
- On Claude Code, use a short `header` chip and put any extra option detail in `description` if the label must stay short.

Keep the full count for the change: 5, 8, or up to 12. The size comes from the scope of the change, never from what fits in one tool call.

If the host caps how many questions a single call may carry, split the quiz across consecutive calls and keep going until all of them have been asked. Claude Code's `AskUserQuestion` takes at most four questions per call, so an 8-question quiz is two calls and a 12-question quiz is three. Do not re-explain the plan between calls, and do not wait for a chat message from the developer to continue.

Do not grade, hint, or comment on correctness until every question has an answer.

If the structured question tool is unavailable, present the same options in chat and accept a plain-language selection. Still do not require letter codes.

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

If a question is skipped, answered with Other, or otherwise incomplete, re-ask the missing items before grading. Do not infer unanswered questions.

### Passing

If every answer is correct:

1. State `Quiz Gate: PASS`.
2. Briefly summarize the areas the developer demonstrated understanding of.
3. Proceed with implementation.

### Blocking

If any answer is incorrect, teach first and ask second. The remediation turn has the same shape as the original gate, including the one-turn rule: explanation as visible text, then the follow-up questions before the turn ends. Do not stop after the explanation to see whether the developer wants to retry.

1. State `Quiz Gate: BLOCKED`.
2. Name each concept the developer missed.
3. Explain that concept properly before asking anything else. Say what the correct understanding is, and why the answer they picked does not hold here. Give the developer enough to reason it out themselves next time, not a one-line correction — this is a second teaching step, not a grading note.
4. Then re-quiz the missed concepts through the same structured question UI, with roughly two or three new questions per missed concept. Several angles on the same idea show whether the explanation landed; a single replacement question can be cleared by a lucky guess.
5. Write genuinely new questions. Change the scenario, the framing, and the options. Never re-ask a question the developer has already seen, and never reuse the option set from a question whose answer key is now known.

Continue this loop until every tested concept has been answered correctly. If the developer misses the follow-ups too, explain the concept again from a different starting point, then ask again with another fresh set.

Do not retest concepts the developer has already passed unless later answers reveal a contradictory understanding.

If a question was ambiguous, withdraw it and replace it. Do not penalize the developer.

If the plan changes during the discussion, update the plan and replace only the questions affected by that change.

## 5. Detect guessing when possible

Multiple-choice answers can be guessed.

If a selected answer conflicts with the developer's accompanying explanation or another answer:

1. Do not pass that concept yet.
2. Point out the apparent contradiction.
3. Ask one short scenario question testing the same concept, in the same turn as that observation.
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
3. Write out the proposed delta. That explanation is how the developer learns the new decision.
4. Open a small Delta Quiz Gate containing 1–3 questions about the new decision, in the same turn as the delta, without waiting for a reply.
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
