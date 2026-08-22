# `/poteto-mode` — a ready-made orchestration stack

> **pstack** is a Cursor plugin. **`/poteto-mode`** is its front door: you state a goal, it picks a playbook, and it calls other skills (`/how`, `/why`, `/architect`, and the rest) when a step needs them.
> The command is **poteto** (the author's handle), not potato.

**Prerequisites:** [01 — Overview](./01-agent-orchestration-overview.md) and [04 — Skills](../04-skills/01-o-que-sao-skills.md). Install the plugin before you try the prompts below.

---

## Why this page exists

Part 2 taught you to design orchestration yourself: a skill for the process, the main agent for edits, sub-agents for isolated work.

**pstack** is someone else's version of that design, packaged as a plugin. You do not have to invent playbooks for "fix a bug with evidence" or "explain how this subsystem works." You install it, then type `/poteto-mode` and a checkable outcome.

Use it when you want a **standard method** across chats. Keep writing your own skills when the process is **yours** (this repo's conventions, your deploy checklist, a team workflow that pstack cannot know).

---

## What pstack is

Four layers sit on top of Cursor:

| Layer | Job |
| ----- | --- |
| **`/poteto-mode`** | Dispatcher. Matches your request to a playbook and stays on for later turns until you opt out or start a new task. |
| **Playbooks** | Named workflows (investigation, bug fix, feature, babysit a PR, overnight runs, and more). You usually do not name them. |
| **Routed skills** | `/how`, `/why`, `/architect`, `/arena`, `/swarm`, `/interrogate`, `/tdd`, `/blast-radius`, and others. Mode calls them. You can also invoke one directly. |
| **Principles** | Short biases such as smallest change, prove it on the real artifact, encode a repeated lesson as a check instead of more prose. |

The author's line: go deep first, write less, verify on the real thing. Throughput without that check is not the goal.

Install from the [Cursor marketplace](https://cursor.com/marketplace/cursor/pstack). Source: [cursor/plugins `pstack/`](https://github.com/cursor/plugins/tree/main/pstack).

---

## What `/poteto-mode` does with your prompt

```
Your prompt
    → poteto-mode reads its principles
    → matches a playbook
    → copies that playbook's steps into the todo list
    → calls /how, /why, /architect, … only when a step needs them
    → verifies and reports
```

You give **intent** and a way to know you are done. The skill carries the **order of operations**.

A skipped step should stay on the list with `skip: <reason>`. If the agent drops steps silently, say so and ask it to follow the playbook.

---

## The loop (the part worth memorizing)

For work that can change the system:

**Understand → design → build small units → prove on the real artifact → encode what you learned.**

| Moment | Typical skill | Job |
| ------ | ------------- | --- |
| You lack a mental model | `/how` | How the subsystem works. Parallel explorers, then one explanation. |
| You might change a historical shape | `/why` | Why it was built that way, from git, tickets, docs, chat, if those tools are connected. |
| The change crosses a module boundary | `/architect` | Types and module shape before filling in bodies. |
| Several valid designs | `/arena` | Same prompt, several models, pick and graft. |
| A diff you do not trust | `/interrogate` | Several models review. The lead judges. Nothing auto-applies. |
| A small-looking change | `/blast-radius` | One safety fact, proven by running code. |
| Done | Your project's tests and checks | "It compiles" is not done. |

Read-only questions skip design and build. That is the **investigation** playbook: `/how`, plus `/why` when the question is motivation.

---

## First setup

1. Add the plugin in Cursor (`/add-plugin pstack`, or install **pstack** from the plugin marketplace).
2. Run **`/setup-pstack`** once. Skills assume model names you may not have. Map each role to a model you can actually spawn, or use `inherit-parent` / `auto`.
3. Install **cursor-team-kit** as well if you want `/deslop` and the CLI/UI control skills that poteto-mode mentions but does not bundle.

Without step 2, multi-model review quietly falls back to one model, and that is most of the product.

---

## How to prompt it

Say the goal and how you will know it worked. Do not list every skill.

**Bug with a check:**

```text
/poteto-mode users get two notifications after a retry. repro first, then fix and verify.
```

**Understand, do not edit:**

```text
/poteto-mode new task. figure out why the cache entry survives logout. don't change any code yet.
```

`new task` tells the sticky mode to re-match instead of continuing the last playbook.

**You are leaving:**

```text
/poteto-mode i'm stepping away. keep going until the migration check reports zero old callers. log your decisions.
```

**Pitfall:** do not write "use /how, then /architect, then /arena." The playbook already sequences those. Name a skill only when you want to **override** a step.

When the chat already has the task, `/poteto-mode do it` or `keep going until done` is enough. The playbook holds the structure.

---

## `/how` and `/why` (when to call them yourself)

Use **`/how`** when you want a walkthrough you will keep: overview, key concepts, how it works, where files live, gotchas.

Use **`/why`** when you are about to change a decision that has history (a magic number, an import exception, a feature flag). It searches source control and whatever MCPs you have (Linear, docs, Slack, Sentry). Empty sources are part of the answer. It is expensive. Do not run it on every file.

Use **`/teach`** when you want how + why as one explanation, diagram by diagram.

For day-to-day implementation, stay on `/poteto-mode` and let it route.

---

## How this fits the rest of the guide

| Guide idea | pstack equivalent |
| ---------- | ----------------- |
| Rule (always on) | Principles + `/setup-pstack` model rule |
| Skill (on demand) | Playbooks and routed skills |
| Main agent | The chat where you typed `/poteto-mode` |
| Sub-agent | Explorers, investigators, arena runners, reviewers |

pstack does **not** replace project skills. If the repo has its own skills or a check command (`make test`, lint, a feature-spec suite), say so in the prompt. poteto-mode does not know those unless you point at them.

---

## When not to use it

- Casual questions ("what does this function return?"). Stay in Ask mode.
- A one-line rename. Smallest change; skip the full feature playbook.
- `/why` as a habit. Use it before you rewrite a boundary, not before every edit.
- Overnight autopilot until your CI and secrets actually work in that environment.

Opt out of the sticky mode by saying so. It should stay out of the way on casual turns.

---

## Try it once

1. Open a real repo in Cursor.
2. Confirm pstack is enabled and `/setup-pstack` has run.
3. Paste:

```text
/poteto-mode new task. explain how the main HTTP entrypoint starts, without changing any files.
```

4. Watch the todo list. You should see investigation-shaped steps, not an immediate refactor.
5. If you then want a change, start with `/poteto-mode new task.` and a **checkable** done state (a test, a command, a screenshot).

---

## Summary

| Idea | One-line reminder |
| ---- | ----------------- |
| **pstack** | Plugin: playbooks + skills + principles |
| **`/poteto-mode`** | Default entry. Goal + check. It routes. |
| **`/how`** | How the code works |
| **`/why`** | Why it was built that way (cited evidence) |
| **Your skills** | Still required for *this* project's process |

---

> **See also:** [01 — Overview](./01-agent-orchestration-overview.md) · [02 — Orchestrating in practice](./02-orquestrando-na-pratica.md) · [pstack in cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack)
