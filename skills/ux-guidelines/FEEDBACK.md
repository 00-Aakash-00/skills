# Feedback and Error Communication

The reference for mutation feedback and error paths. Loaded when wiring any mutation or writing any user-facing error. The surface decision table and the message anatomy live in [SKILL.md](SKILL.md); this file deepens them without redefining them.

## Significance scaling

Three tiers calibrate how loud feedback should be:

- **Micro** — the state change is the confirmation. The card sits visibly in the Done column; the deleted row is gone; the toggle is on. Add nothing on top.
- **Standard** — the result isn't where the user is looking, or nothing visibly changed. Confirm inline, or with a toast when the surface that triggered the action is gone.
- **Rare** — genuine milestones: the first publish, a completed onboarding, a long-earned goal. Celebration is allowed here precisely because it is rare; daily confetti is wallpaper.

Tiebreaker: when unsure whether an action needs feedback, it does.

Done when: every mutation in the change is assigned a tier.

## Choosing the surface: worked examples

**Inline** — the default.
- A field error renders under the field it belongs to.
- A section's fetch error renders inside that section's own card.
- A failed action's "Try again" renders beside the control that failed — the user's eyes are already there.

**Toast** — safe to miss only.
- "Couldn't connect — retrying" while a background retry runs.
- "Saved" when a sheet closed itself on success and there is nowhere inline left to say it.
- Disqualified: anything about money, and anything the user must act on — if missing the message would harm the person, it isn't a toast.

**Modal** — blocking, with the way forward.
- Payment method expired mid-checkout: block, explain, and put "Update payment method" in the modal.
- Access revoked on the open document: block and offer "Request access".
- Disqualified: anything the user could reasonably defer — the way forward is what earns the interruption.

**Self-closing containers.** A dialog or sheet that closes on success unmounts its own inline feedback. Either confirm before the close, or hand the confirmation to a surface that survives the unmount (usually a toast). The success message the system composed must reach the user, not the void.

Done when: every mutation's success and failure each have a named surface, chosen by the table, that provably outlives the interaction.

## Error message anatomy: worked examples

The formula — what happened + why (when known) + a clear next action — applied per failure class:

| Raw (never ship) | Shipped |
|---|---|
| `duplicate key value violates unique constraint "users_email_key"` | That email is already registered. Sign in instead, or use a different email. |
| `Something went wrong` (on a payment) | Your card wasn't charged. The payment service didn't respond — try again, or use a different payment method. |
| `TypeError: Failed to fetch` | Couldn't reach the server. Check your connection and try again. |
| `Request failed with status 500` | We couldn't save your changes — nothing was lost. Try again in a moment. |

Log the raw detail server-side with a reference id; show the human message, and include the reference id where support might need it. Payment-grade and data-loss-grade errors state the outcome for the user's money or data ("your card wasn't charged", "your draft is safe") — that outcome is the fact the person is anxious about.

Voice and tone of error copy belong to the frontend-design skill; this file owns the structure.

Done when: every shipped message carries all three parts of the anatomy, and every money or data error states the outcome.

## The silent-failure audit

The silent failure — the action that fails and tells no one — hides inside code that looks defensive. Audit for it:

1. **List** every site in the touched code where a failure can vanish: catch blocks (empty ones above all), fallbacks that return an empty collection or null on error, discarded promise or action results, fire-and-forget calls, and success paths that close the UI before confirming.
2. **Classify** each site into exactly one of:
   - *Surfaces* — the failure reaches the user through a surface from the table.
   - *Justified silence* — the failure is genuinely inconsequential to the user, and a written comment at the site says why.
   - *Bug* — fix it now.
3. **Reconcile optimism.** Optimistic updates that apply state before the server confirms must roll back on failure and tell the user. An optimistic update without a rollback path is a silent failure with extra steps.

Done when: every listed site is classified, zero remain unclassified, and no classification reads "probably fine".
