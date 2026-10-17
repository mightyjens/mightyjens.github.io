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

Communication runs through my own **Matrix server**; tasks and code live in my **Gitea instance**.

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

1. Create issues in Gitea, set a priority.
2. Send `!plan` in Matrix. The Scrum Master posts a sprint proposal and one card per issue.
3. Approve a card with ✅ (or `!approve all`, `!reject`, `!retry`, `!pause` …).
4. The orchestrator queues the issue, starts a fresh sandbox and lets the coding agent ([Aider](https://aider.chat), headless) work against the LLM.
5. Green tests → PR with `Closes #<nr>` and the test output, linked in the card's thread. Red tests → `WIP:` PR and `blocked`.
6. I review and merge – the orchestrator sets `done`.

A nice side effect: if I reply in a card's Matrix thread, it ends up as a comment on the issue, and the agent reads it on its next run. That lets me steer after a `blocked` without opening Gitea. Via an `#inbox` room, spontaneous thoughts are turned into issue drafts by the LLM, which I move into the backlog with ✅.

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

- **The LLM backend is the bottleneck.** My LocalAI processes requests one after another. While the coding agent waits for a response, `!plan` and the inbox hang as well. A non-reasoning model for small tasks (effort S) cut the runtime from up to 10 minutes to under one.
- **Long silence breaks log streams.** The log stream through the socket proxy dropped during minutes of waiting, and the run was wrongly reported as failed even though the PR was there. Fix: reconnect and check the end state via a status query.
- **Agents don't automatically read project rules.** Aider doesn't pick up `AGENTS.md` on its own; the LLM only sees the prompt, the repo map and the files in the chat. That's why the wrapper writes the rules into the prompt itself.
- **Vague issues create long loops.** "Check the tests again" is not a task. The title must describe the outcome, the description the acceptance criteria. One issue equals exactly one well-defined task.
- **Never push to an already merged PR branch.** Follow-up commits got lost that way – since then I've enabled "Delete branch after merge".

## Current Status and Outlook

Version 1 runs end to end: planning, approval, implementation in the sandbox, PR, merge. Open items include:

- CI via `act_runner` as a status check in branch protection,
- an iteration limit and a token budget,
- monitoring of the health endpoint,
- sandboxes for more languages (currently only Python with `pytest`; C#, Node and AL are still missing),
- moving the LLM backend to an NVIDIA DGX Spark.

My main takeaway so far: with tight guardrails, small issues and a human reviewing every PR, a local agent team is already usable today – for boilerplate, small features and refactorings. It does worse on long, multi-step tasks, and there the only remedy is: cut the task smaller.
