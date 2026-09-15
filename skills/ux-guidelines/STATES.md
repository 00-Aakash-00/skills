# States, Loading, and Graceful Degradation

The construction reference for the four states and section independence. Loaded when work touches a data-backed screen or section, a known-slow operation, or background work. The doctrine and the loading ladder live in [SKILL.md](SKILL.md); this file deepens them without redefining them.

## Section independence patterns

A section is any region of a page with its own data source: a stats row, a feed, a notifications panel, a settings card. Give each one:

- **Its own fetch.** The section's data arrives independently; no other section waits for it.
- **Its own boundary.** A failure inside the section is caught at the section's edge and rendered inside the section's own frame.
- **Its own retry.** The error state includes a control that refetches only this section, in place, without a full page reload.

The blast radius of any failure stays one section.

The anti-pattern is the aggregate: one request (or one parallel batch awaited as a unit, or one catch wrapped around the lot) feeding N sections. Structurally:

```
Aggregate (avoid):                    Independent (target):
fetch A+B+C as one unit               section A: fetch A -> render A, or error A + retry A
  -> render all three, or             section B: fetch B -> render B, or error B + retry B
  -> one error/blank for all three    section C: fetch C -> render C, or error C + retry C
```

One slow query in the aggregate blanks all three sections; one failure takes down the page.

**The mixed-state wireframe.** For each page, answer on paper: which sections can fail independently, and what does the page look like in each failure that matters? Enumerate at least every one-section-down combination. Each failed section renders an in-place error card of the same size and position as its healthy self, so the layout holds.

Done when: every section on the page can fail alone while the rest stays interactive, and each single-section failure has a designed in-place rendering.

## Stale-while-revalidate

When a screen has shown data before, show it again immediately:

1. Render the last known good content on arrival — no loader.
2. Refresh in the background.
3. Swap in the fresh data without a layout jump when it arrives.
4. If the refresh fails, keep the stale content and add an unobtrusive "couldn't refresh" indicator with a retry — stale-and-honest beats blank.
5. Show a freshness timestamp only where data age changes decisions (dashboards, monitoring); omit it where it is noise.

Done when: no refresh path can blank a screen that has ever shown data.

## Loading: skeletons and spinners

Skeletons hold space for layout regions — cards, lists, tables, charts. Mirror the real structure: same heights, same widths, same row count, so the swap to real content is a fill-in rather than a reflow. A skeleton that doesn't match its content is a second layout shift. Spinners mark small contained work: a button submitting, a control refreshing.

Both obey the ladder's first rung. Mark loading regions with `aria-busy="true"` and `role="status"` so assistive tech hears the load instead of silence.

Done when: every skeleton visually matches the content it stands in for, and every loading region is announced.

## Empty: three kinds

An empty state answers "why is there nothing here, and what can I do about it?" Three kinds, three answers:

- **First-use** — the user hasn't created anything yet. An invitation: say what belongs here and offer the create action as the CTA.
- **Cleared** — the user finished everything. A confirmation, often a positive one ("You're all caught up") — this emptiness is success, so render it as success.
- **Filtered** — a search or filter matched nothing. Say so, and offer to clear the filter or broaden the search.

Render emptiness explicitly, always: a section that silently disappears when its list is empty is indistinguishable from a section that broke. For the voice and visual direction of empty-state copy, see the frontend-design skill.

Done when: every list and section has an explicit empty rendering of the right kind, with a CTA wherever an action exists.

## Error and success, honestly

Section-scoped errors follow the anatomy in [SKILL.md](SKILL.md) (worked examples in [FEEDBACK.md](FEEDBACK.md)) and carry the section's retry.

Guard the line between error and empty in the data layer: a fetch that returns an empty collection on failure makes the UI tell the user "create your first item" over a broken backend — an error wearing an empty state's clothes. Failures propagate as failures, so the section boundary catches them; empty results return as empty results. The two must be distinguishable at the call site.

Done when: no code path converts a failure into an empty collection or a null that the UI then renders as legitimate absence.

## Long operations

Past ~5 seconds, narrate. When the system has real stages, name them ("Uploading photo" → "Analyzing" → "Saving") — staged text that changes buys patience that a looped animation costs. Rotate messages every few seconds even when the stages are synthetic; movement in the text reads as progress. Text carries the wait up to about 10 seconds; past that, provide:

- a progress bar or step indicator — the structure that shows real advancement, which text loops can no longer fake,
- a cancel affordance wherever abandoning is safe,
- the error the moment failure is known — never spinner-then-fail.

Done when: no operation in the flow can show an unchanging loader for more than ~5 seconds, or a text-only one for more than ~10.

## Background work

Work the user didn't initiate — sync, recomputes, scheduled refreshes — still gets a surface: a last-updated stamp, a subtle refreshing indicator, or both. When a background job fails, the screen says so ("Updated 3 hours ago — refresh") rather than silently presenting stale numbers as current. Stale-and-honest beats silently wrong.

Done when: every background process that feeds visible data has a visible freshness or failure affordance.
