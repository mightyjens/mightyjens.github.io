---
#slug: agenten-devteam-homelab
title: 'My Agent Dev Team in the Homelab: Gitea, Matrix and a Sandbox for AI-Generated Code'
date: 2026-10-17T09:00:00+02:00
draft: false
#author: ["mightyjens"]
description: "How I built a self-hosted AI dev team made of a Scrum Master and a Developer agent – controlled via Matrix, managed in Gitea, isolated in a gVisor sandbox."
categories: ["coding"]
tags: ["AI","Homelab","Agents","Gitea","Matrix","Docker","Proxmox","Python"]
series: [""]
categories_weight: 10
#cover:
#  image: posts/agenten-devteam-homelab/cover.png
#  caption: ""
#  hiddenInList: true
comments: true
showReactions: 1
showToc: true
TocOpen: false
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
UseHugoToc: true
---

I wanted to find out how far you can get with a self-hosted AI development team – without an agent running amok on my network. The result is a small system of two agents that I control via chat and whose output I review the classic way: as a pull request. Here is the architecture and the most important learnings.

## The Idea

Two roles, just like in a real Scrum team:

- **Scrum Master agent:** gives me an overview of open issues and bugs, estimates type and effort (S/M/L), proposes an order and asks for my approval.
- **Developer agent:** implements approved issues on its own – branch, code, tests, pull request.

Communication runs through my own **Matrix server**; tasks and code live in my **Gitea instance**. A dedicated **inbox channel** in Matrix lets me create simple issues in Gitea straight from the chat – without ever opening the web UI.

## Four Architecture Principles

1. **Gitea is the single source of truth.** The agents don't talk to each other. All state lives in issue labels, comments and PRs. Matrix is just the remote control.
2. **One central orchestrator** runs as a state machine (Python, `matrix-nio`) and reacts to Matrix events and Gitea changes.
3. **Human in the loop at two points:** approving tasks in Matrix, and PR review plus merge by me in Gitea. The agent never merges on its own.
4. **Swappable chat adapter** behind a small interface, so Matrix can be replaced later.

## The Workflow

Every issue passes through mutually exclusive status labels:

```
backlog → proposed → approved → in-progress → review → done
                                    └─→ blocked (error, red tests, timeout)
```

In daily use it looks like this:

1. Create issues in Gitea, set a priority – or simply drop an idea into the Matrix inbox channel (see below).
2. Send `!plan` in Matrix. The Scrum Master posts a sprint proposal and one card per issue.
3. Approve a card with ✅ (or `!approve all`, `!reject`, `!retry`, `!pause` …).
4. The orchestrator queues the issue, starts a fresh sandbox and lets the coding agent ([Aider](https://aider.chat), headless) work against the LLM. The LLM can be local or in the cloud, with LocalAI acting as a proxy in between. That way different models can be mapped to different effort levels (S/M/L).
5. Green tests → PR with `Closes #<nr>` and the test output, linked in the card's thread. Red tests → `WIP:` PR and `blocked`.
6. I review and merge – the orchestrator sets `done`.

`!plan` in Matrix: the Scrum Master answers with a sprint proposal and posts one card per issue, which I approve with ✅:

![Matrix: the `!plan` command returns a sprint proposal with the recommended order and an issue card with ✅/❌ reactions](/posts/2026/agent-devteam-homelab/matrix-plan-proposal.png)

Once approved, the Dev Agent takes over. It starts the run, opens the PR with green tests and asks me to review – after the merge, the Scrum Master closes the loop:

![Matrix: the Dev Agent starts the run on branch agent/issue-48, reports PR #49 with green tests, and the Scrum Master confirms the merge](/posts/2026/agent-devteam-homelab/matrix-approved-pr-merged.png)

The same issue as seen in Gitea – the full workflow is visible in its timeline. The Scrum Master sets the labels and assigns the Developer agent, which links its PR and commit; the `status` label moves from `approved` via `in-progress` and `review` to `done` after my merge, and the run result (PR, test status, duration, model, log file) is posted as a comment:

![Gitea issue timeline: scrum-bot sets type, prio, source, effort and status labels, dev-bot is assigned and references PR #49, the status label moves from approved via in-progress and review to done](/posts/2026/agent-devteam-homelab/gitea-issue-timeline.png)

A nice side effect: if I reply in a card's Matrix thread, it ends up as a comment on the issue, and the agent reads it on its next run. That lets me steer after a `blocked` without opening Gitea.

### The Inbox Channel: Issues Straight from Matrix

Not every idea deserves a trip to the Gitea web UI. That's why there is a dedicated `#inbox` room in Matrix. I write a short, free-form message there – "add a `--dry-run` flag to the export script", say – and the orchestrator has the LLM turn it into a clean issue draft with title and description. Once I confirm the draft with ✅, the issue is created in Gitea and lands in the backlog, ready for the next `!plan`.

![Matrix inbox: a free-form message becomes an issue draft with context, goal and acceptance criteria; after ✅ the issue is created in Gitea](/posts/2026/agent-devteam-homelab/matrix-inbox-issue-draft.png)

This is meant for **simple issues** only: small, clearly scoped tasks that I can describe in a sentence or two. Anything more complex still gets written up properly in Gitea, because vague issues create long loops (more on that below). The big advantage is that the barrier to capturing a thought is almost zero – I can do it from my phone while walking the dog, and it ends up in the same single source of truth as everything else.

## Labels: Type, Priority and Effort

Besides the `status/*` labels, every issue carries three more label groups. All of them are mutually exclusive – one label per group – and they are what turns a plain backlog into something the Scrum Master can plan with.

### Types

| Label | Meaning |
|---|---|
| `type/bug` | Something is broken and should behave differently. |
| `type/feature` | New behavior or capability. |
| `type/chore` | Maintenance work without visible behavior change: refactoring, cleanup, dependencies. |

The type is optional. If I don't set it, the Scrum Master has the LLM estimate it during `!plan` and sets the label itself – but it never overwrites a label I set by hand.

### Priority

`prio/high`, `prio/medium` and `prio/low` decide the order in the queue. An issue without a priority simply goes last, which is a good incentive to set one.

### Efforts

| Label | Typical task | Model used |
|---|---|---|
| `effort/S` | Small, clearly scoped change (a flag, a one-line fix, a small helper) | fast model without reasoning |
| `effort/M` | A normal feature or bug fix touching a few files | default model |
| `effort/L` | Larger change across several modules | default or a stronger model |

Effort is also optional and is estimated by the LLM during `!plan` if missing, again without overwriting. But it is more than a label: the effort decides **which model does the work**. Each size can be mapped to its own model via configuration (`LLM_MODEL_S`, `LLM_MODEL_M`, `LLM_MODEL_L`), falling back to a default model. Because LocalAI sits in front as a proxy, a "model" is just an alias – it can point to a local model or to a cloud API.

This turned out to matter a lot in practice. My default model reasons by default, which is great for hard problems but wasteful for small ones. For `effort/S` I mapped a second alias to a non-reasoning model, and a typical small run dropped from up to ten minutes to about 20 seconds.

Effort also acts as an early warning: if an issue looks like an `L` and keeps hitting the 30-minute limit, it's too big. The fix is never a longer timeout, but splitting it into smaller issues.

### What the Card Shows

All of this ends up on the issue card in Matrix, in one line – for example `🐞 bug · 🔴 prio high · ⏱ effort M · 📌 proposed` – followed by a short summary. Fields that are missing are simply hidden. So when I approve with ✅, I see at a glance what kind of work it is, how urgent it is and roughly how big it is.

## Where Does It All Run?

Orchestrator, sandboxes and (planned) the Gitea Actions runner all run together in **one dedicated Proxmox VM** – deliberately not an LXC and not in the Docker network of Gitea or Synapse:

- The orchestrator needs Docker access (effectively root), and the sandboxes execute LLM-generated code. That shouldn't share a blast radius with my Gitea or Matrix instance.
- A VM is a stronger boundary than an LXC, and snapshots and backups apply only to this VM: if an agent makes a mess, only this VM gets rolled back.

```
agent-vm (Docker)
├── socket-proxy       → only place with docker.sock, exec blocked
├── orchestrator       → Python, Matrix bot, state machine, worker, webhook
├── sandbox-<issue>    → per task, gVisor (runsc), dedicated network
└── act_runner         → CI, planned
```

## Hardening: Don't Trust the Agent

The core of the project is less the agent than the question of how to set limits for it:

- **gVisor (`runsc`)** as the runtime for every sandbox container.
- **Docker socket proxy:** The orchestrator talks to Docker only through a proxy that blocks `exec`. Since the proxy filters endpoints but not contents, the sandbox parameters are hard-wired in the orchestrator code.
- **Dedicated sandbox network:** outbound traffic only to Gitea and the LLM endpoint. No LAN, no internet.
- **Resource limits and capabilities:** CPU, RAM and PID limits, `--read-only`, `--cap-drop=ALL`, `no-new-privileges`.
- **Secrets invisible to generated code:** token and LLM key are removed from the environment at startup, processes are set to non-dumpable, foreign processes are killed before the push.
- **Branch protection as a requirement:** Before every run, the orchestrator uses the bot token to verify that the default branch is actually protected – otherwise nothing starts (fail closed).
- **Diff check after the run:** exactly one commit on top of the recorded base commit, no changes under `.gitea/` or `.github/`. The orchestrator code itself also lives deliberately outside the org the agents work in – no agent can modify its own supervisor.
- **Hard limits:** 30 minutes per task, 5 minutes per LLM call.

## The Gitea Webhook as an Alarm Clock

The orchestrator polls Gitea. An org webhook merely wakes this poller up immediately – deliberately not a second state machine. The payload isn't evaluated; the poller reads fresh state from the API. If the webhook fails, the regular polling interval simply kicks in. That keeps the system robust and simple.

## Learnings

- **Agents don't automatically read project rules.** Aider doesn't pick up `AGENTS.md` on its own; the LLM only sees the prompt, the repo map and the files in the chat. That's why the wrapper writes the rules into the prompt itself.
- **Vague issues create long loops.** "Check the tests again" is not a task. The title must describe the outcome, the description the acceptance criteria. One issue equals exactly one well-defined task.
- **Never push to an already merged PR branch.** Follow-up commits got lost that way – the fix is to enable "Delete branch after merge" in Gitea.

## Current Status and Outlook

Version 1 runs end to end: planning, approval, implementation in the sandbox, PR, merge. Open items include:

- CI via `act_runner` as a status check in branch protection,
- an iteration limit and a token budget,
- monitoring of the health endpoint.

My main takeaway so far: with tight guardrails, small issues and a human reviewing every PR, a local agent team is already usable today – for boilerplate, small features and refactorings. It does worse on long, multi-step tasks, and there the only remedy is: cut the task smaller.
