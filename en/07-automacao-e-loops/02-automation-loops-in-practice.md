# Automation loops in practice — step by step

> Define the exit condition first. Then choose local loop, Cloud Agent, or Automation.

---

## When to set up a loop

Do this when:

- You have already fixed the **same class of failure** more than once (CI, lint, flaky tests)
- A process has a clear **“done” signal** (checks green, checklist complete)
- You want the agent to **verify its own work** before asking you to review
- The team needs the **same loop** on every PR or every week

---

## Step by step: local CI fix loop

### 1. Define the exit condition in one line

**Example:** “All GitHub Actions checks on this PR pass.”

That line becomes the skill’s **Exit condition** section and what you repeat in the prompt.

---

### 2. Create the loop skill

Create `.cursor/skills/loop-on-ci/SKILL.md` (or your project’s name for the same idea):

```markdown
# Loop on CI

> Fix failing checks until green or max iterations reached.

## Trigger

- CI failed, checks red, "fix CI", "get green"

## Iteration steps

### 1. List failures

Run or read CI output. One bullet per failing check with the error snippet.

### 2. Fix highest-signal failure

Smallest change that addresses root cause. One category per iteration when possible.

### 3. Verify and commit

Re-run the check locally if possible. Commit with message referencing the check name.

## Exit condition

- All required checks pass, OR
- 5 iterations reached → stop, list remaining blockers, ask human

## Human checkpoint

- Review diff before push/merge
```

Commit the skill so the team shares the same loop.

---

### 3. Run the loop in Agent mode

Open a session on the failing branch. Keep context minimal: the skill reference, branch name, and CI log if needed.

**Example prompt:**

```
Use the loop-on-ci skill in .cursor/skills/loop-on-ci/SKILL.md.

Fix failing checks on this branch. After each attempt, show:
1. What failed
2. What you changed
3. Whether checks pass now

Stop at 5 iterations or when all checks are green.
```

---

### 4. Review as the human orchestrator

Before merge:

- [ ] Fixes match the failures (no unrelated refactors)?
- [ ] Exit condition was actually verified (not assumed)?
- [ ] Commits are readable for reviewers?

---

## Step by step: review-and-ship loop

Use this when code is mostly done but you want the agent to **iterate on quality** before you open or update a PR.

### 1. Write the ship criteria

Example checklist:

- Tests pass locally
- Lint clean
- No debug logging left
- PR description filled in
- Scope matches the ticket

Put the checklist in `.cursor/skills/review-and-ship/SKILL.md`.

### 2. Run with explicit loop instruction

```
Use review-and-ship skill. Work through the checklist.
For each item that fails, fix and re-check.
Stop when all items pass or after 3 full passes — then summarize what's left.
```

### 3. Optional: sub-agent for exploration

If failures need wide repo search (e.g. “find all call sites”), use a **read-only sub-agent** first, then feed the summary to the main agent for edits. Same pattern as [06 — Orchestrating in practice](../06-orquestracao-de-agentes/02-orquestrando-na-pratica.md).

---

## Step by step: Cloud Agent for a long task

### 1. Write a complete brief

Cloud Agents work best when the task is **self-contained**:

- What to build or fix
- Where in the repo (paths, patterns)
- How to verify (tests, commands)
- What is out of scope

Attach **rules** and **skills** in the repo so the cloud environment picks them up.

### 2. Start a Cloud Agent

- In Cursor: select **Cloud** next to the agent input
- Or: [cursor.com/agents](https://cursor.com/agents)

Paste the brief. Mention relevant skills by path if the task matches a playbook.

### 3. Review artifacts before merge

When the agent finishes, check:

- PR diff and description
- Attached logs, screenshots, or videos
- Whether verification steps actually ran

Take over the remote desktop if you need to click through the UI yourself.

---

## Step by step: create an Automation

### 1. Pick the trigger

| Goal                         | Trigger to consider              |
| ---------------------------- | -------------------------------- |
| Review every new PR          | GitHub/GitLab: PR opened         |
| React to CI failures         | GitHub/GitLab: CI completed      |
| Weekly chore                 | Scheduled (cron)                 |
| Triage from chat             | Slack: New message               |
| Custom integration           | Webhook                          |

Create at [cursor.com/automations/new](https://cursor.com/automations/new) or in the **Agents Window**.

### 2. Write the automation prompt

Be specific about:

- What to check, change, or produce
- Which tools are enabled (comment on PR, Slack, MCP)
- When to **act** vs. **do nothing** (quality bar)

**Example (PR review automation):**

```
When a non-draft PR opens on this repo:

1. Read the diff and linked issue if present
2. Check: tests added for new logic, no secrets, matches .cursor/rules/
3. If issues found: leave inline comments on the PR with file/line
4. If critical security issue: request review from @security-team
5. If looks good: leave a short approval comment summarizing what was reviewed

Do not merge. Do not push commits unless the PR author asked for fixes.
```

### 3. Choose repository access

- **No repository** — Slack/MCP/webhook workflows only; no code edits
- **Single repository** — review or fix one codebase
- **Multi-repo environment** — tasks that span multiple codebases

### 4. Activate and monitor

- Run once manually if the UI allows a test fire
- Check the first few runs for noise (false positives)
- Tighten the prompt when the agent over-acts or under-acts

Browse [Automations marketplace](https://cursor.com/marketplace/automations) for templates (PR review, Slack digest, security scan).

---

## Organizing loop skills in the repo

Prefer **small, named loops** over one “fix everything” skill:

```
.cursor/skills/
├── loop-on-ci/           ← checks green
├── review-and-ship/      ← pre-PR quality pass
├── fix-flaky-test/       ← one test file, iterate
└── create-vue-component/ ← one-shot (not a loop)
```

**Composable:** orchestration skills call one-shot skills inside a loop (e.g. review-and-ship runs lint skill each iteration).

---

## Testing a loop

1. **Start with a known failure** — red CI or failing test on a throwaway branch
2. **Cap iterations low** (3–5) on first run
3. **Require evidence** — agent must paste command output or check status
4. **Compare** — run the same task once without the skill; note drift and extra prompts
5. **Update the skill** when you discover a missing step or a false “done”

---

## Full example (story)

**Scenario:** PR opened with two failing checks — lint and unit tests.

1. **Local loop (author’s machine):** developer runs `loop-on-ci` skill in Agent mode; agent fixes lint, then tests, in separate iterations; pushes when green.
2. **Automation (team):** “PR opened” automation runs a readonly review; comments if new code lacks tests.
3. **Cloud Agent (optional):** for a large fix the author delegates “fix all test failures on branch `feature/x`” to Cloud; reviews PR when notified.

Same **rules** apply everywhere. **Skills** define how each loop behaves. **You** merge.

---

## Summary

1. Write the **exit condition** before the prompt.
2. Capture repeatable local loops in **`.cursor/skills/`** with max iterations and verification.
3. Use **Cloud Agents** for long, well-scoped tasks you can review asynchronously.
4. Use **Automations** for team-wide, event-driven loops.
5. Always keep a **human checkpoint** before merge or production.

---

> **Next:** [07 — Exercise: automation loops](../05-guias-praticos/07-exercicio-automation-loops.md) — build and run a loop skill on a real project · [06 — Agent orchestration](../06-orquestracao-de-agentes/01-agent-orchestration-overview.md) — skills, agents, and sub-agents
