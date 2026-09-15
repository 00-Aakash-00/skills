# Forms

A form's job is to make success obvious and failure recoverable. Frustration compounds per field: each unclear requirement, surprise rejection, or lost keystroke multiplies across everything the user already typed. Every section below ends on its own bar; the checklist at the bottom aggregates them.

## Submission gating

Keep submit disabled until the form is valid — and make what's missing really obvious: required fields are marked, incomplete fields are identifiable, and the reason the button is disabled is visible next to it ("Pick at least one weekday"). A greyed-out button with no explanation is more frustrating than a rejected submit; the disabled state only works when the user can see exactly why.

Escape hatch: when validity is too complex to signal passively, an always-enabled submit that validates on press, shows every error inline, and moves focus to the first invalid field is the honest alternative.

Bar: a user can always answer "why can't I submit?" without guessing.

## Inline validation on field exit

Validate each field when the user leaves it — the moment they believe they're done with it. Per-keystroke validation shouts at half-typed input (an email address is invalid until its last character); submit-only validation batches every punishment for the end and sends the user scrolling back to fix things. Once a field has shown an error, re-validate on change so the error clears the instant it's fixed — a stale error on a now-valid field reads as a broken form.

Bar: an invalid field announces itself before submit, and a corrected field clears itself immediately.

## Live counters near limits

Where a field has a length limit, show a live count as the user types — counting down, flagging overflow before submit. Let no one write a paragraph and then learn they must delete half of it.

Bar: submit is never the first place a limit is discovered.

## Prefill what you know

Signed-in users never retype what the system knows: email, name, saved addresses, locale-appropriate defaults (country, currency, units, date format). Choose defaults that are right for most users and let the rest change them.

Bar: no field asks for information the system already holds.

## Live requirement check-offs

Constrained fields — passwords above all — list their requirements up front and check each off live as it's satisfied. The anti-pattern is requirements revealed one rejection at a time: the user learns "needs a capital letter" only by failing it.

Bar: every constraint is visible before the first submit attempt, and its state updates as the user types.

## Forgiving formats

Accept every unambiguous way a human writes the value: phone numbers with dashes, parentheses, dots, spaces, or nothing; dates in local conventions; numbers with or without separators. Normalize to canonical form server-side. Reject only genuinely unparseable input — and then say what form you expected.

Bar: no format-only rejection of parseable input.

## Field errors

Field errors render inline, under or beside their field, wired with `aria-invalid` on the field and `aria-describedby` pointing at the message so assistive tech reads them together. The message follows the anatomy in [FEEDBACK.md](FEEDBACK.md), scaled down: what's wrong and what to enter instead. The error clears when the field becomes valid.

Bar: every field error is adjacent to its field, announced to assistive tech, and self-clearing.

## Long forms

Past roughly seven fields, split the form into steps — multi-step forms have shown conversion lifts around 300% over single-page monoliths. Show progress ("step 2 of 4"), group related fields per step, and persist partial input across steps and reloads so a stumble never costs the whole form. This is Hick's law applied to fields; the general law lives in [CHOICES.md](CHOICES.md).

Bar: no single screen asks more than ~7 fields, and no navigation or refresh loses entered data.

## Form checklist

- [ ] The user can always answer "why can't I submit?"
- [ ] Invalid fields announce on exit and clear on fix.
- [ ] Limits show live counters before they bite.
- [ ] Nothing the system knows is asked again.
- [ ] Constraints are visible up front and check off live.
- [ ] Parseable input is never rejected for format.
- [ ] Field errors are inline, announced, and self-clearing.
- [ ] Long forms are stepped, with progress and persistence.
