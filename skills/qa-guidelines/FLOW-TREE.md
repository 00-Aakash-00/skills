# The Flow Tree

The reference for steps 2 and 3: mapping the branching tree of user flows, driving it like a human, and reporting the result. Loaded when the QA agent plans its coverage or writes its report. The doctrine lives in [SKILL.md](SKILL.md); this file deepens it without redefining it.

## Mapping

1. **List the roots.** Every entry point a user can arrive through: the landing page, logged-in home, the logged-out state, first run and onboarding, a deep link, an email or notification link, the return from an external step (OAuth callback, payment redirect), a shared URL opened cold.
2. **At each node, list every action.** Every clickable, every form with its submit and its cancel, every navigation, every keyboard shortcut, every gesture on mobile, the browser or OS back button, refresh, and time itself (session expiry, an app backgrounded and resumed).
3. **Each action is an edge to a child node.** Follow it to the resulting screen or state and repeat.
4. **Terminate a branch** when the user's goal is reached, the user has exited, or the edge lands on a node already in the tree — mark the loop rather than re-expanding it.
5. **Prune only in writing.** Every branch left unmapped carries a reason in the report ("admin-only, no admin credentials"; "identical to the sibling branch above"). An unwritten prune is a gap.

Scope by the work: for a change, root at the changed surface and include every flow that leads into it and every flow it leads out to — the neighbors are where regressions hide. For a release, map from every root.

Done when: the tree is written down before the first action is taken, and every prune has its reason beside it.

## Notation

An indented tree. Each line is a node reached by an action, followed by its status once driven:

```
Root: /login (logged out, cleared storage)
├─ valid credentials → /dashboard                          pass
├─ wrong password → inline error, email kept               pass
├─ submit with both fields empty → "Email is required"     FAIL #1
├─ Enter in password field → same as submit                pass
├─ "Forgot password" → /reset
│  ├─ known email → "Check your inbox"                      pass
│  ├─ unknown email → same message, no enumeration         pass
│  └─ browser back → /login, fields empty                  pass
├─ refresh mid-typing → fields cleared, no error           pass
└─ session expired on /dashboard → back to /login, banner  blocked (needs expiry under 24h)
```

Statuses: **pass**, **FAIL #n** (pointing to a finding), **blocked** (unreachable because of an earlier failure or a missing precondition, with the reason), **not reached** (in the tree, not driven — with the reason).

## The un-happy branch catalog

At every node, ask which of these edges exist, and drive each one that does:

- Cancel or dismiss — from a dialog, a sheet, a multi-step flow.
- Back — browser back, Android back, the app's own back — from every step, including the last.
- Refresh or reload — mid-form, mid-upload, on a results page.
- A deep link straight to this node with no prior state.
- Empty required input; whitespace-only input.
- Invalid format; boundary lengths (one character, the maximum, one past it); pasted content.
- Double-click or double-tap on submit; a submit during a pending submit.
- Slow network; a failed request; offline, then back online.
- No data yet (a new account); one item; a thousand items.
- Expired session; logged out in another tab; insufficient permission.
- A second tab or device on the same account, acting at the same time.
- A phone width; keyboard-only navigation.

Done when: every node in the tree has been checked against the catalog, and every catalog edge that exists is in the tree.

## The human moves

A script walks the shortest path and never varies it. A person does none of that, and the difference is where bugs live. While driving, deliberately:

- Hesitate on a step, then continue — do timeouts or auto-saves fire?
- Change your mind halfway: fill three fields, leave, come back.
- Type the wrong thing, see the error, fix it — does the error clear? Does the fix stick?
- Use back instead of the app's own navigation, then forward again.
- Open the same screen in two tabs and act in both.
- Resize the window from desktop to phone width with a form half-filled.
- Kill the network mid-action; restore it.
- Do things out of the intended order: jump to step three by URL, submit before a required upload completes.
- Read every message as the person would: is it clear what happened and what to do next?

Done when: each root flow has had at least one human move applied on every step that accepts input.

## Traversal rules

- Fresh session per root; one branch at a time; no state shared between roots.
- Observe after every action before acting again: the screen, what changed, the console, the network.
- Expectation comes from what a reasonable person would expect at this node, and from the intent in the brief. Where a screen's loading, error, and empty behavior is in question, the ux-guidelines skill is the bar when installed.
- Record a failure with its evidence and keep going. Mark downstream branches that depend on the failed step as blocked.
- A branch passes only when the visible outcome matched expectation *and* nothing went wrong beneath it.
- Never fix. The QA agent that finds a bug reports it; the implementer fixes it; a new QA pass verifies the fix.

## Severity

- **Blocker** — a flow cannot complete, data is lost or corrupted, money moves wrongly, or a security or privacy boundary breaks.
- **Major** — a wrong outcome with a workaround, a silent failure, a misleading message on a consequential action.
- **Minor** — cosmetic, copy, or a rough edge that doesn't change the outcome.

## The report

1. **Header** — surface, driver, build (URL, commit, or version), accounts and data used, date, viewports.
2. **The tree** — with a status on every line.
3. **Findings** — one per failure: id, severity, the branch path, exact steps and inputs, expected, actual, evidence.
4. **Coverage** — counts by status, the not-reached and blocked lists with reasons, and every UI bypass used.
5. **Not driven** — anything the brief asked for that no available driver could reach, and what would unblock it.
6. **Other test runs** — unit and integration results, reported separately.

Done when: an implementer can reproduce every finding from the report alone, and a reader can see at a glance which branches were never driven.

## Re-verification

After fixes, a new QA pass — in a subagent — drives every failed branch, its parent, and its siblings, plus any branch the fix could plausibly touch. A branch that passed before and fails now is a regression and is reported as one. Repeat until the tree has no fails and no unexplained blocks.
