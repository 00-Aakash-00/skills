---
name: ux-guidelines
description: Behavioral UX guidelines — the four screen states (loading, error, empty, success), graceful degradation, action feedback, and forgiving forms. Use when building or changing any user-facing surface (a page, screen, component, dialog, or flow), wiring data fetching or loading states, adding a mutation or action button, building or fixing a form, writing error handling or user-facing error messages, laying out navigation or option-heavy screens, making a touch surface feel native on mobile, reviewing an app's UX, or when an app "feels broken" or fails silently. Covers behavior only; for visual direction, typography, and interface copy see the frontend-design skill, and for UI polish and animation see the emil-design-eng skill.
---

# UX Guidelines

UI is whether the interface looks good. UX is whether a person can figure out what to do — and what happens to them when something fails. When a button does nothing or behaves unpredictably, the user blames themselves, and that broken trust is why people leave. Generated code defaults to the happy path: the screen as it looks when every request succeeds, every list has items, and every field is valid. This skill exists to make you build the un-happy paths unprompted: the four states of every screen, feedback for every action, and failure that degrades gracefully instead of silently. Treat everything below as the bar for "done", not as polish to add later.

## Companions

UI and UX are separate passes on the same work: this skill owns behavior — states, feedback, degradation, forms. Whenever you build or reshape UI, also load the **frontend-design** skill (aesthetic direction, typography, and interface copy, including the voice of error and empty-state copy) and the **emil-design-eng** skill (component polish and animation) if they are available in this project or globally. When reviewing animation or motion code specifically, use **review-animations** if available. If a companion is not installed, proceed — this skill is self-sufficient for behavior; do not reconstruct the missing skill's content from memory.

## The four states

Every screen has four states — **loading**, **success**, **error**, and **empty** — and so does every section within it that fetches its own data. Success is the only state you get for free; the other three exist whether or not you design them, and an undesigned state is a design decision made by the runtime: a blank region, a stack trace, a spinner that never resolves. Design each one deliberately:

- **Loading** — what holds the space while data is on its way (see the loading ladder below).
- **Success** — the data, rendered.
- **Error** — what the person sees when the fetch fails, and how they retry.
- **Empty** — what the screen says when the fetch succeeds and there is nothing to show.

A surface is done when you can point at all four states and say where each renders. Construction depth — skeletons, the three kinds of empty, the errors-as-empty trap, long operations — is in [STATES.md](STATES.md); read it whenever the work touches a data-backed screen or section.

## Section independence

Graceful degradation is a structural property, not an error-handling afterthought: each section of a page owns its data, its loading state, its error state, and its retry. The blast radius of one failed fetch is one section — the page shell, the navigation, and every healthy section stay rendered and usable. Aggregating a page's fetches into one all-or-nothing request (or one shared catch) couples every section to the slowest and least reliable one.

Design the mixed state explicitly: "what does this page look like when some sections work and others don't" is a question you must be able to answer with a sketch, not a shrug. And when a section has shown data once, keep showing it: serve the last known good content immediately, refresh in the background, and swap when fresh data arrives — stale-while-revalidate. A screen that had data never goes blank.

Patterns, the aggregate anti-pattern, and the mixed-state exercise are in [STATES.md](STATES.md).

## Feedback for every action

Every mutation tells the user three things: it started (pending), then it either worked or it didn't. Uncertainty is worse than failure — "did my payment go through?" is a worse state than a clear decline, because the person can't even decide what to do next. Scale feedback to significance: often the state change itself is the confirmation (the card now sits in Done; the row is gone), a routine save earns a small confirmation, and celebration is reserved for rare milestones — confetti fired daily stops meaning anything.

Choose the surface with this table:

| Surface | Use when | Rule |
|---|---|---|
| **Inline** (default) | The message belongs to a specific control or section | Render it next to the thing — the field error under the field, "Try again" beside the failed action. Closer is better. |
| **Toast** | The message is safe to miss | Auto-dismisses. Never the sole carrier of information the user must act on. |
| **Modal** | The user genuinely cannot continue | Must include the way forward — a control that resolves the blockage — never just "OK". |

Escalate only as far as needed: the least interruptive surface that still guarantees the message lands. Feedback must also survive whatever closes or unmounts around it — a dialog that closes on success still owes the user its confirmation, delivered by a surface that outlives the dialog.

Worked examples, significance tiers, and the silent-failure audit are in [FEEDBACK.md](FEEDBACK.md); read it whenever you wire a mutation or write an error path.

## Error messages

A good error message says three things: **what happened**, **why** (when known), and **a clear next action**. "Your payment didn't go through — your card was declined. Check your card details or try a different payment method." Write the human message; raw backend, database, or exception text never reaches a user — it is unreadable to the person it's shown to and a security disclosure of your internals to everyone else. Hold payment-grade and data-loss-grade actions to the highest bar: they never get a bare "something went wrong", because the user needs to know what happened to their money or their data.

The worst error is the silent failure — the button that does nothing, the delete that fails but closes the dialog anyway. It is the default output of generated code, so hunt for it deliberately; the audit method is in [FEEDBACK.md](FEEDBACK.md).

## The loading ladder

Loading treatment is a function of expected wait time:

| Expected wait | Show |
|---|---|
| Under 1 second | Nothing — delay the loader (200ms–1s) so operations that finish fast never flash one. A spinner flash on a fast operation reads as a glitch and makes the app feel slower. |
| 1–5 seconds | A skeleton for layout regions; a spinner for buttons and small controls. |
| 5–10 seconds | The loader plus a label — static ("Loading", "Saving") buys a little patience, text that changes ("Connecting to your account" → "Almost there") buys significantly more. |
| 10+ seconds | A progress bar or step indicator — something that shows real advancement. Text loops and looped animations stop carrying the wait past ~10 seconds and actively erode patience. |
| On failure | The error, immediately. The loader never outlives the failed request — spinner-then-fail after twenty seconds is a double failure. |

Skeletons hold layout so the brain pre-processes structure before data arrives; spinners mark small, contained work. Construction rules — and the treatment of known-slow operations and background work — are in [STATES.md](STATES.md).

## Reduce choices, follow conventions

Decision time grows with the number and complexity of visible options (Hick's law), and people expect your app to work like the rest of the web they already know (Jakob's law). Load [CHOICES.md](CHOICES.md) when a screen offers many options, when designing navigation or menus, or when the surface spans desktop, mobile, or RTL locales.

## Forms

Make it obvious what's missing, and be forgiving about format. When building or touching any form, load [FORMS.md](FORMS.md) — it carries the frustration reducers (validation timing, counters, prefill, requirement check-offs, forgiving formats, long-form splitting), each with its own completion bar.

## Mobile feel

Touch is not hover, and mobile browser chrome makes viewport heights lie. When a surface ships to phones — touch interactions, app-like layout, a notch to respect, or "feels like a website" complaints — load [MOBILE.md](MOBILE.md). Its bar is a pass on a real phone, not in desktop emulation.

## Build checklist

Run this list before calling any user-facing work done. When reviewing existing UI, run it as an audit and report each unchecked item as a finding with file and line.

- [ ] Every screen and every independently fetched section is accounted for across all four states. Produce the surface-by-state table; done means no cell reads "whatever the framework does".
- [ ] Every mutation has pending, success, and failure feedback — and that feedback survives unmounts (a sheet closing on success, a navigation away).
- [ ] Zero silent failures: every catch block, error-swallowing fallback, and discarded action result either surfaces to the user or carries a written justification for staying silent.
- [ ] No raw backend, database, or exception text reaches a user; every payment-grade or data-loss-grade error states what happened to the user's money or data.
- [ ] Every failed section has a retry that refetches only that section; the page shell and healthy sections render regardless of any one section's failure.
- [ ] Every async wait maps to a rung of the loading ladder, and on failure the loader is replaced immediately.
- [ ] Every form answers "why can't I submit?" at a glance, and parseable input is never rejected for format alone.
- [ ] Errors never masquerade as empty states — an empty screen means the request succeeded and returned nothing.
- [ ] Option-heavy screens are curated — a few choices plus filters, disclosure, or defaults — and every layout follows its platform's conventions unless the deviation is deliberate.
- [ ] Surfaces that ship to phones pass the mobile bar on a real device: no stuck hover, no tap flash, no zoom-on-focus, no hijacked scroll, no content trapped by the notch.
