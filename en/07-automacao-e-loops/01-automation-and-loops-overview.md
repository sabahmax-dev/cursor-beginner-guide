# Automation and loops — when the agent keeps going

> **Skill** documents the process. **Loop** repeats it until a condition is met.
> Automation hands the repetition to the agent — with guardrails.

---

## What are automation loops?

An **automation loop** is a workflow the agent runs **more than once** until something measurable is true: tests pass, CI is green, a review checklist is complete, or a PR is ready to merge.

This is different from the **Research → Plan → Implement** loop in [01 — Foundations](../01-fundamentos/03-workflow.md). That loop is **human-driven** — you move between modes on purpose. Automation loops are **agent-driven** — you define the exit condition and let the agent iterate.

Think of it in three layers:

| Layer            | Who drives it | Typical scope                          |
| ---------------- | ------------- | -------------------------------------- |
| **Local loop**   | You + agent   | One session: fix → test → fix again    |
| **Cloud Agent**  | Agent         | Long task in the cloud; you check back |
| **Automation**   | Event/schedule| Runs without you starting each time    |

When these layers line up with **skills**, **rules**, and **orchestration**, you stop retyping the same “try again until it works” prompt.

---

## How this relates to what you already know

| Concept          | Role in a loop                                              |
| ---------------- | ----------------------------------------------------------- |
| **Rules**        | Boundaries that never change (style, security, stack)       |
| **Skill**        | Defines each iteration’s steps and validation checklist     |
| **Orchestration**| Who explores, who edits, who summarizes between iterations  |
| **Loop**         | Repeats the skill until the exit condition is met           |

```
Rule (always-on) → Skill (each iteration) → Agent (acts)
                        ↑
            Sub-agent (optional: explore failures / run checks)
                        ↓
              Exit condition met? ──no──→ loop again
                        │
                       yes
                        ↓
              Human checkpoint (merge, deploy, approve)
```

> Skills: [04 — Skills](../04-skills/01-o-que-sao-skills.md) · Orchestration: [06 — Agent orchestration](../06-orquestracao-de-agentes/01-agent-orchestration-overview.md)

---

## Three kinds of automation

### 1. Local iterative loops (IDE)

You stay in Agent mode and ask the agent to **keep going** until a condition is true. The agent runs commands, reads output, fixes code, and retries.

**Best for:** fixing failing tests, getting lint green, tightening a PR before review.

**Guardrails you must set:**

- **Exit condition** — what “done” means (e.g. `npm test` exits 0)
- **Max iterations** — stop after N tries and ask for human help
- **Scope** — which files or checks the agent may touch

**Example prompt:**

```
Use the loop-on-ci skill. Fix failing checks on this branch.
Stop when all CI checks pass or after 5 iterations — whichever comes first.
Show me what failed before each fix attempt.
```

### 2. Cloud Agents (background agents)

**Cloud Agents** run in an isolated VM in the cloud. They plan, edit, run commands, test their work, and can open a PR — without your laptop staying connected. They were formerly called Background Agents.

**Best for:** features that take a long time, bug fixes you want to start and review later, work you can describe clearly up front.

**How to start:**

- In Cursor, select **Cloud** next to the agent input
- On the web or mobile: [cursor.com/agents](https://cursor.com/agents)
- From Slack, GitHub, or Linear: mention `@Cursor`

Cloud Agents produce **artifacts** (logs, screenshots, videos) so you can validate work without checking out the branch.

> Details: [Cloud Agents — Cursor Docs](https://cursor.com/docs/cloud-agent)

### 3. Automations (always-on)

**Automations** run Cloud Agents on a **schedule** or when an **event** fires — PR opened, CI completed, Slack message, Linear issue created, webhook, and more.

**Best for:** PR review on every open, triaging bugs from Slack, security scans after CI, scheduled cleanup jobs.

**How they work:**

1. Choose a trigger (schedule, GitHub PR opened, CI completed, etc.)
2. Write a prompt telling the agent what to do
3. Select tools (comment on PR, Slack, MCP, memories)
4. Attach zero, one, or multiple repositories
5. Save and activate

Automations do not require you to be at the keyboard. Each run is billed at API pricing for the selected model.

> Details: [Automations — Cursor Docs](https://cursor.com/help/ai-features/automations.md) · Templates: [Automations marketplace](https://cursor.com/marketplace/automations)

---

## When to use loops

| Situation                         | Typical approach                          |
| --------------------------------- | ----------------------------------------- |
| Fix CI until green on your branch | **Local loop** + loop skill               |
| Review PR and ship when ready     | **Local loop** + review-and-ship skill    |
| Implement a feature while away    | **Cloud Agent**                           |
| Review every new PR automatically | **Automation** (PR opened trigger)        |
| Triage bugs from Slack            | **Automation** (Slack message trigger)    |
| Weekly dependency cleanup         | **Automation** (scheduled trigger)        |

**Rule of thumb:** if you have said “try again until it works” more than twice on the same task, capture it as a **loop skill** or an **automation**.

---

## Anatomy of a loop skill

Loop skills live under `.cursor/skills/` like any other skill, but they add **iteration** and **stop conditions**:

```markdown
# Loop on CI

> Fix failing checks until green or max iterations reached.
> Use when CI fails on your branch or PR.

## Trigger

- "Fix CI", "get checks green", "loop until CI passes"

## Preconditions

- [ ] On the correct branch
- [ ] Can run the same checks locally (or via CI logs)

## Iteration steps

### 1. Read failures

- Run checks or read CI output
- List each failing check with the error message

### 2. Fix one category at a time

- Pick the highest-signal failure first
- Make the smallest fix that addresses the root cause

### 3. Verify

- Re-run the failing check locally if possible
- Commit with a descriptive message

## Exit condition

- [ ] All required checks pass
- OR max iterations (default: 5) reached → stop and summarize blockers

## Human checkpoint

- [ ] Review the diff before merge
- [ ] Confirm no unrelated changes slipped in
```

---

## Local loop vs. Cloud Agent vs. Automation

| Aspect              | Local loop              | Cloud Agent              | Automation                    |
| ------------------- | ----------------------- | ------------------------ | ----------------------------- |
| **You start it**    | Yes, each session       | Yes, once per task       | No — trigger starts it        |
| **Runs while away** | No (session-bound)      | Yes                      | Yes                           |
| **Best iteration**  | Tight feedback loops    | Long, end-to-end tasks   | Repetitive team-wide tasks    |
| **Cost model**      | Your IDE session        | Per Cloud Agent run      | Per automation run            |
| **Human review**    | Between iterations      | Before merge             | Before merge (or auto-comment)|

---

## Common mistakes

| Mistake                          | Effect                              | Fix                                           |
| -------------------------------- | ----------------------------------- | --------------------------------------------- |
| No exit condition                | Agent loops forever, burns context  | Define “done” + max iterations in the skill   |
| No verification step             | Agent claims success prematurely    | Require command output or CI status in skill  |
| Automating unclear work          | Noisy PRs, wrong fixes              | Start with local loops; automate when stable  |
| Skipping human checkpoint        | Bad code merged                     | Always review before merge                    |
| One mega-loop for everything     | Hard to debug failures              | Small loop skills per scenario (CI, tests, lint)|

---

## Summary

| Concept           | One-line reminder                                           |
| ----------------- | ----------------------------------------------------------- |
| **Local loop**    | Agent retries in your session until exit condition or cap   |
| **Loop skill**    | Documents iteration steps, verification, and stop rules     |
| **Cloud Agent**   | Long-running agent in the cloud; reports back with artifacts|
| **Automation**    | Cloud Agent on a schedule or event — no manual start          |
| **Human checkpoint** | You approve before merge, deploy, or production changes  |

---

> **Next:** [02 — Automation loops in practice](./02-automation-loops-in-practice.md) — step-by-step: CI loop, review-and-ship, Cloud Agents, and Automations
