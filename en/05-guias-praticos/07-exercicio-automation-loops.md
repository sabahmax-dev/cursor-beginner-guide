# Exercise 7: Automation loops in practice

> **Goal:** Create and run a loop skill that iterates until a clear exit condition is met.
> **Mode:** Agent Mode
> **Estimated time:** 25 minutes
> **Prerequisite:** [04 — Skills](../04-skills/01-o-que-sao-skills.md) and [07 — Automation and loops](../07-automacao-e-loops/01-automation-and-loops-overview.md)

---

## Scenario

You will create a **loop skill** (not a one-shot skill), run it on a real check in your project, and observe how exit conditions and max iterations keep the agent on track.

---

## Exercise 7.1 — Create a loop skill

We will create a skill to fix failing checks until green or until a max iteration count.

### Prompt to copy and paste:

```
Create a loop skill at .cursor/skills/loop-on-ci/SKILL.md

The skill should document an iterative process to fix failing CI or local checks
on the current branch. Base verification commands on what this project actually uses
(test runner, linter, typecheck).

The skill must include:
1. Trigger — when to use this skill
2. Preconditions — branch, ability to run checks
3. Iteration steps — read failures, fix one category, verify, commit
4. Exit condition — all checks pass OR max 5 iterations
5. Human checkpoint — review diff before merge

Include concrete commands for this repository (read package.json, Makefile, or CI config).
```

### What to watch for:

- The agent should **inspect how checks run** in your project before writing the skill
- The skill should require **evidence** (command output), not just “I fixed it”
- Max iterations must be explicit

---

## Exercise 7.2 — Run the loop (simulated or real)

If you have a failing check, use it. If everything is green, introduce a small safe failure (e.g. unused import that fails lint) on a throwaway branch.

### Prompt to copy and paste:

```
Use the loop-on-ci skill in .cursor/skills/loop-on-ci/SKILL.md.

Fix failing checks on this branch. After each iteration show:
1. What failed (with command output snippet)
2. What you changed
3. Whether checks pass now

Stop at 5 iterations or when all required checks are green.
```

### What to watch for:

- The agent loads the skill and follows **iteration steps in order**
- Each iteration ends with a **verification** attempt
- The agent **stops** when green or at the cap — it does not loop forever

### Verify:

- [ ] Skill was loaded?
- [ ] Failures listed with evidence?
- [ ] Fixes are scoped (no drive-by refactors)?
- [ ] Loop stopped at exit condition or max iterations?

---

## Exercise 7.3 — Compare: one-shot fix vs. loop

Run the same failure once **without** referencing the loop skill — only “fix the failing check.”

| Aspect              | Without loop skill        | With loop skill              |
| ------------------- | ------------------------- | ---------------------------- |
| Prompt size         | Longer each time          | Short reference to skill     |
| Verification        | Often skipped             | Required each iteration      |
| Stop condition      | Vague                     | Explicit cap + green checks  |
| Repeatability       | Drifts between sessions   | Same steps every time        |

---

## Exercise 7.4 — Optional: sketch an Automation

You do not need to activate billing or production automations for this exercise. Write the prompt you **would** use if a “PR opened” automation reviewed every PR in this repo.

### Prompt to copy and paste:

```
Draft an automation prompt (do not create it in cursor.com) for:
Trigger: GitHub PR opened on this repo.

The automation should:
- Read the diff
- Check against our .cursor/rules/ if they exist
- Comment on the PR if tests are missing for new logic
- Do nothing if the PR looks fine

Output the prompt text only, suitable for cursor.com/automations/new.
```

### What to watch for:

- Clear **when to act vs. do nothing**
- No merge or push unless explicitly allowed
- References real project standards

---

## What we learned

1. **Loop skills add iteration and stop rules** — not just ordered steps
2. **Exit conditions must be verifiable** — command output or CI status
3. **Max iterations protect context and cost** — always set a cap on first use
4. **Local loops compose with Cloud Agents and Automations** — same skills, different triggers
5. **Human checkpoint stays mandatory** — loops automate repetition, not approval

---

## Congratulations!

You completed all exercises. You now know how to:

- **Research** with Ask Mode before acting
- **Plan** with Plan Mode before implementing
- **Implement** with Agent Mode in a controlled way
- **Use rules** for automatic consistency
- **Create skills** to automate repetitive tasks
- **Orchestrate** skills, main agent, and sub-agents
- **Run automation loops** with guardrails until work is truly done
- **Manage context** to keep answer quality high

### Next level

- Add a `review-and-ship` loop skill for pre-PR quality passes
- Try a **Cloud Agent** on a well-scoped task at [cursor.com/agents](https://cursor.com/agents)
- Browse [Automations marketplace](https://cursor.com/marketplace/automations) for team templates
- Experiment and iterate — that is how you learn

> _“Pick one tool and practice a lot. It is part art and part science — you need repetition to develop intuition about what to trust the model with and what to keep for yourself.”_

---

> **See also:** [07 — Automation and loops](../07-automacao-e-loops/01-automation-and-loops-overview.md) · [06 — Full workflow](./06-exercicio-workflow-completo.md)
