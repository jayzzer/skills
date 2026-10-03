---
name: iterative-loop-contract
description: >-
  Step contract for implementation work without an upfront plan: every step is
  understand → act → inspect → clarify → adjust and ends in a runnable check. Use for any code
  change, including one-step fixes (they take a short path), to size steps, decide which decisions to make silently, announce, or ask about, pivot
  when the approach is wrong, keep tests as the feedback loop, and log contract changes between
  parallel agents. Not for explaining code, searching code, or reviewing a diff.
---

# Iterative Loop Contract

Do not write or wait for approval of an implementation plan. Frame the task, then move in small checked steps. Planning happens inside the loop; the human stays oriented through the decisions the loop surfaces, not through a document.

## Task Frame

Before the first step, hold three short lines:

- **Goal:** the observable outcome.
- **Done when:** verifiable criteria (tests, behavior, command output).
- **Out of scope:** plausible work that must not be done.

If the task does not state them, derive them from evidence and record them as assumptions. Ask only when they cannot be derived and a wrong guess is expensive to undo.

Problems noticed along the way that the task does not ask about (a clumsy neighbour, a tempting refactor) belong to **Out of scope**: name them in the result as follow-ups, do not fix them.

A change that fits in one step still follows this contract on a short path: no frame and no step names; make it, run its check, report.

Do not front-load clarifying questions. Start the first step in the same reply. Questions come later, from what the steps reveal (see **Decisions**).

The one exception is an ask-first change: a data schema or migration, a public API or protocol (`.proto`, OpenAPI, shared types), a dependency, security behavior, CI or deploy config, an irreversible operation, or a VCS mutation. Ask about it before touching it, even in the first reply and even when the task itself requests it (see **Decisions** for what to ask).

## Step Contract

Every step has five parts:

1. **Understand** — one or two lines: what is known now and what this step changes. Name the step as `<change> → <check>`; a step without a named check is not a step. Reading code is groundwork, not a step.
2. **Act** — one small change: a function and its test, a component and its render test, a mapper and its call site. Not "the feature".
3. **Inspect** — the narrowest check that proves the step: targeted test, typecheck, lint, build, or running the code. A step that cannot be checked is too large; split it.
4. **Clarify** — classify the decisions this step took or revealed (see **Decisions**): make them silently, announce them, or stop and ask.
5. **Adjust** — act on the check result and any answer before the next step: fix, narrow, or pivot.

Example split for "add sorting by rating": (1) comparator and its unit test; (2) the new option in the sort control and a render test; (3) wire the comparator into the list and run the affected tests. Each step is green before the next begins.

Do not start the next step while the current check is red, unless the failure is proven to be pre-existing or environmental; report that distinction.

## Decisions

Every choice the task does not settle falls into one of three tiers. The deciding factor is how much later work builds on the choice, not how uncertain it feels.

1. **Silent** — only one reasonable option, or the task, existing code, or conventions settle it: naming, file location, following an established pattern. Just do it.
2. **Announce** — a real alternative exists, but changing it later costs no more than redoing the current step. Take it and write one line at the moment of the decision, in the user's language: `Decision: X over Y, because Z.` Keep working. The user sees these lines while the work runs and may redirect; treat such a message as input to **Adjust**.
3. **Ask** — stop and ask when both hold: the user could reasonably choose differently (product behavior, UX, a business rule, an architecture boundary such as where state lives or how modules split), and later steps would build on the choice, so changing it later means redoing more than the current step. If evidence in the code or the task settles it, it is not a question.

How to ask:

- Not upfront. First do the groundwork and the cheapest step that makes the question concrete ("the badge now uses the median of the current page; should it use the whole result set?"), then stop before building on it.
- Batch the pending questions into one stop. For each: the options, the recommended one first, what is already done, and what depends on the answer. Use a structured question tool if one is available.
- Do not keep building on the unanswered choice.

Independently of the tiers, ask **before acting** on changes that alter contracts or are hard to undo:

- data schemas and migrations;
- public APIs, protocols, and shared types consumed outside the package;
- dependencies (added, removed, or version changes);
- auth, permissions, secrets, and other security behavior;
- CI, build, and deploy configuration;
- deletion of data or other irreversible operations;
- VCS mutations: commit, push, PR publication.

A task that itself requests such a change authorizes the change, not its shape. Before editing, propose the concrete shape (names, types, versioning, migration path) and get it confirmed; groundwork such as reading the schema may come first.

## Tests As Feedback

- Change code and its test in the same step. For new behavior, write the failing test first.
- Red → fix → rerun is the default loop. Escalate only when the loop is stuck.
- Never weaken or rewrite existing tests silently. Skipping, focusing, deleting assertions, loosening expectations, or updating snapshots requires first establishing that the new behavior is intended, and then naming the test change and its reason in the result. "Make CI green" is not a reason.
- A check that could not run is not a pass. Report the command, the error, and the residual risk.

## Pivot, Do Not Replan

Stop and report instead of continuing when:

- the same check fails three times with different fixes;
- evidence refutes the understanding the work started from;
- the next step turns out to need an ask-first change that was not anticipated.

Report what was learned, why the current approach is wrong, the proposed alternative, and what decision is needed. Do not silently rewrite the goal or expand the scope.

## Zoom Out

Small green steps can still drift. Before finishing, and whenever the change grows beyond a few steps, check the diff against **Done when** and **Out of scope**, and revert work that no longer serves the goal.

## Result

Apart from decision lines and questions, no per-step narration. The final report contains:

- outcome against each **Done when** criterion;
- checks run, with exact commands and results;
- decisions worth reviewing: the announced ones, each with the alternative it rejected;
- questions still open, if any;
- residual risk.

Do not retell the diff.

## Parallel Agents: Decision Registry

When several agents work on one change, share a registry of contract changes instead of a plan.

- Fix shared contracts (interfaces, schemas, props, API shapes) in one step before splitting work. The registry records changes; it does not resolve conflicts after the fact.
- Append one line per contract change: `date | agent/branch | contract | change | reversible: yes/no`.
- Read the relevant entries before starting and before touching a shared contract.
- Keep the registry outside the task diff, at the path the orchestrator provides.

## Precedence

A stricter repository or preset policy (for example, scope, worker, or validation contracts) takes precedence over this one. This contract governs step size, feedback, and when to ask; it does not grant additional scope.
