---
name: qa-guidelines
description: Human-simulating end-to-end QA — every QA pass runs in an independent subagent that drives the app's real surface (web, iOS, Android, desktop, CLI, agent-facing API) through the full branching tree of user flows, instead of trusting the unit and integration tests the implementing agent wrote to pass. Use when a feature, fix, or refactor is "done" and needs verification, when asked to test, QA, verify, smoke-test, or "check that it works", before a PR, release, or deploy, after any change to a user-facing flow, when reviewing an app end to end, or when testing an AI product (chatbot, agent, copilot, LLM or RAG feature) — where the only valid check is simulating users and reading the answers and traces, never keyword or regex assertions. Covers the QA process only; for the behavior a surface should have (states, feedback, forms) see the ux-guidelines skill.
---

# QA Guidelines

The agent that wrote the code also wrote its tests, so a green run proves the code does what that agent thought — not what a person needs. Those tests were shaped to pass, they cover the cases their author thought of, and they miss every case the author didn't. A test suite is a treadmill: put the car on it, wire a computer to the steering, repeat the same three movements, and the wheels turn perfectly while every real road condition goes untested. QA is letting a driver take the car out. This skill makes you do that: hand QA to an independent agent, pick the surface a human actually uses, map the branching tree of flows through it, and drive every branch the way a person would. That drive is the bar for "done"; the implementer's green run is not.

## The rule: QA runs in an independent subagent

Whenever work is ready to be called done — a feature, a fix, a refactor, a release — spawn a separate agent to QA it. In Claude Code that is the Agent tool; in Codex, Cursor, and other harnesses, a subagent or a fresh session. Never QA your own change inside the context that produced it: the implementer knows the intended path and walks it, knows which cases it skipped and steers around them, and reads its own output as correct. A fresh agent has only the brief and the app, so it finds what is actually there.

The brief hands the QA agent:

- **The surface and how to reach it** — URL, build, simulator, or binary, plus the test accounts and data it may use.
- **What a user should now be able to do** — the intent of the change in user terms. Not the diff, and not the implementer's tests: the QA agent may read code to map flows, but its verdict comes from driving the surface.
- **The scope** — which flow roots to cover (see step 2), and whether this is change-depth or release-depth.
- **This skill** — tell it to load `qa-guidelines` and follow it.
- **The report format** — the tree with statuses plus findings, as in [FLOW-TREE.md](FLOW-TREE.md).

Separation of duties holds throughout: the QA agent reports and never fixes. The implementer fixes, then QA runs again — in a subagent — over the affected branches and their neighbors. If the harness cannot spawn an agent, run QA in a fresh session that receives only the brief. If even that is impossible, the report says so; a self-check is never labeled QA.

## Step 1: the surface

What will a human — or an agent — use your app on? A web app, an iPhone app, an Android app, a desktop app, a CLI, an API consumed by other agents. Coding agents now have tools to drive every one of them. Pick the tool that sees what the user sees and acts where the user acts:

| Surface | Drive it with |
|---|---|
| Web app | A real browser first — Claude in Chrome, Codex's in-app browser — then Playwright CLI or Playwright MCP |
| iOS app | The Simulator (`xcrun simctl`) with a UI driver: Maestro, XCUITest, or a mobile MCP |
| Android app | The emulator with `adb` or Maestro |
| Desktop app | OS-level computer-use or accessibility automation; Playwright for Electron |
| CLI | A real terminal session — run the commands exactly as a user types them |
| Agent-facing API or MCP | A real client, with only the docs and tool descriptions a consuming agent would have |

Interact through the surface, never around it: no `page.evaluate` to set state, no direct database write to skip a signup, no calling the handler instead of clicking the button. A change to shared code is tested through every surface that reaches it. Per-surface mechanics, evidence capture, and what to do when no driver exists are in [SURFACES.md](SURFACES.md).

## Step 2: the flow tree

Within the surface, every decision a user can make — a different button, another page, cancel instead of confirm, back instead of next — branches a new branch off the user flow tree. Map that tree before driving it: roots are entry points, nodes are screens or states, edges are the actions available there. Include the un-happy edges the happy path skips: cancel, back, refresh, wrong input, empty data, expired session, missing permission, offline, double submit.

Scope the tree to the work: for a change, root it at the changed surface plus every flow that leads into or out of it; for a release, the whole app. Write the tree down — it is the test plan going in and the coverage record coming out. The mapping method, notation, and branch catalog are in [FLOW-TREE.md](FLOW-TREE.md).

## Step 3: drive every branch like a human

Put the two together: the QA agent takes the mapped tree and drives every branch on the real surface, simulating a person. Instead of running tests it wrote to pass, it has the full journey of what a user might do and the tools to do it. The rules of the drive:

- **Fresh session per root.** Cleared storage, a new login, the state a real user would arrive with.
- **Look after every action.** What is on screen, what changed, what the console and network said. The expectation is what a reasonable person would expect here, and the intent in the brief — never "what the code does".
- **Make the human moves.** Hesitate, go back, refresh mid-form, double-click submit, paste, leave a field empty, type something too long, resize to a phone width, open a second tab, lose the network. A script never does these; a person always eventually does.
- **Don't stop at the first bug.** Record it with evidence and continue. A branch that can't be reached because of an earlier failure is *blocked*, not passed.
- **Pass means nothing went wrong.** The visible result matched expectation *and* no console error, failed request, or layout break happened underneath.

What a screen owes its user in its loading, error, and empty states, and how an action should confirm itself, is the bar set by the **ux-guidelines** skill — load it when it is installed and judge against it; otherwise judge as a careful person would. The output of the drive is the tree with every branch marked pass, fail, blocked, or not reached, and a findings list with a repro path, expected, actual, and evidence for each failure. Traversal rules, the human-moves catalog, severity, and the report format are in [FLOW-TREE.md](FLOW-TREE.md).

## Unit and integration tests still matter

They are not the enemy — they speed up development, and they are the cheapest way to keep a regression from creeping in as the codebase grows. Keep writing them, keep them green, and report their run alongside QA. What they cannot do is stand in for the drive: the treadmill checks that the wheels turn; only the road checks that the car drives. Both, always; neither substitutes for the other.

## AI products

An AI product — a chatbot, an agent, a copilot, any LLM- or RAG-backed feature — gets the same method with one hard rule on top: **never validate model output with keyword, substring, or regex checks.** Outputs are non-deterministic and paraphrased, so a match proves a string appeared rather than that the answer is right, a miss fails a correct answer worded differently, and an implementer writes checks that match its own prompt's phrasing — overfitting with a different face. The only valid check is the QA agent simulating users: send the messages a real user would send, read the full answer, read the trace — tool calls with their arguments and results, retrieved context, intermediate steps, errors, latency — and judge whether the answer is correct, complete, grounded in that trace, and appropriate.

The conversation is the surface and every user turn is a branch, so the flow tree includes the vague question, the follow-up that depends on context, the correction, the off-topic ask, the adversarial one, and the question whose honest answer is "I don't have that". Run the branches that matter more than once, because a pass on one run in three is a fail. The rubric, the probe catalog, and how to read a trace are in [AI-PRODUCTS.md](AI-PRODUCTS.md).

## QA checklist

Run this before calling any work done. When auditing someone else's QA, run it against their report and treat each unchecked item as a finding.

- [ ] QA ran in an independent subagent that received the brief — surface, user intent, scope, report format — not the diff or the implementer's tests as the definition of correct.
- [ ] The surface driven is the one humans (or consuming agents) use, through a real driver; every bypass of the UI is named in the report.
- [ ] The flow tree is written down, rooted at every entry point in scope, and every branch carries a status: pass, fail, blocked, or not reached.
- [ ] Un-happy branches were driven — cancel, back, refresh, invalid and empty input, empty data, expired session, missing permission, offline, double submit — not only the happy path.
- [ ] Every failure has a repro path, expected, actual, severity, and evidence (screenshot, console, network, or transcript and trace).
- [ ] For AI products: zero keyword or regex assertions; every verdict cites what the answer said and what the trace showed.
- [ ] The QA agent fixed nothing; every fix was re-verified by a new QA pass over the affected branches and their neighbors.
- [ ] Unit and integration tests also ran, and their result is reported separately from — never instead of — the drive.
