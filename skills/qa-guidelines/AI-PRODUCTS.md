# QA for AI Products

The reference for testing chatbots, agents, copilots, and any LLM- or RAG-backed feature. Loaded when the surface under test produces model output. The rule — no keyword or regex validation, ever — lives in [SKILL.md](SKILL.md); this file says what to do instead.

## Why string checks are invalid

Model output is non-deterministic and paraphrased. A substring or regex match proves that a string appeared, not that the answer is right; a miss fails an answer that was correct in different words; and the implementer writes checks that match the phrasing its own prompt produces, which is the test-written-to-pass problem in a new costume. There is no shortcut through the words. The QA agent is itself a reader: it sends what a user would send, reads what came back, reads how it was produced, and judges.

## The conversation is the surface

Every user turn is a branch. Roots are the first messages real users send, and the tree grows with each reply. Map it with the notation in [FLOW-TREE.md](FLOW-TREE.md), with the un-happy branches of conversation in place of cancel and back:

- The core intent, phrased the way a real user types — short, imperfect, missing context.
- A vague version of it ("can you help with the thing from yesterday").
- A follow-up that only makes sense with the earlier turns in mind.
- A correction ("no, I meant the other account").
- Several questions in one message.
- A question whose honest answer is "I don't have that" or "there is no data for this".
- Out of scope, and adversarial: prompt-injection attempts, requests to ignore instructions, requests the product should refuse.
- Empty, a single emoji, a pasted wall of text, another language.
- A long session: twenty turns in, does it still remember turn two?

Drive them as a user: type into the product's actual interface, wait for the whole answer (streaming complete), and read what the user is shown — not only what the API returned.

## Reading the trace

For every turn, find the trace and read it: which tools were called, with what arguments, what they returned, what context was retrieved, what intermediate steps ran, what errored or retried, how long it took, and what it cost if visible. Traces live in the product's observability tool (Langfuse, LangSmith, Braintrust, an OpenTelemetry backend — whatever the project uses), in application logs, or in the agent harness's own transcript. If there is no trace to read, that is the first finding: an AI product whose behavior cannot be inspected cannot be QA'd, and cannot be debugged in production either.

## The rubric

Judge each turn on all of these, and write the judgment as a sentence that cites the answer and the trace:

- **Correct** — the facts match the source of truth: the tool results, the retrieved documents, the data the product owns.
- **Complete** — every part of the question is answered; a two-part question with a one-part answer is a fail.
- **Grounded** — every claim is supported by something in the trace. A claim that appears nowhere in the retrieved context or tool results is a likely hallucination; report it as one, quoting the sentence and naming the absence.
- **Right tools** — the right tool, with the right arguments, and its result actually used. A tool that should have been called and wasn't, a result that was ignored, and a made-up result are each findings.
- **Behaved** — stays in scope, refuses what it should and only that, keeps the product's tone, and never reveals what it shouldn't (the system prompt, other users' data, internal ids).
- **Recovered** — when a tool fails or returns nothing, the answer says so honestly and offers a next step, rather than inventing a result or going silent.
- **Consistent** — the same question asked again gets a materially equivalent answer; follow-ups keep the thread's context; a correction is honored on the next turn.
- **Usable** — latency a person would tolerate, streaming that doesn't stall, and the loading, error, and empty states any surface owes its user.

## Verdicts

A verdict is never "contains X". It is: what the user sent, what the answer said, what the trace shows, and therefore the judgment.

```
Turn 3 — user: "what did I spend on ads last month?"
Answer: "$4,210 across Meta and Google."
Trace: search_transactions(category="advertising", month="2026-08") → 6 rows totaling $3,980; no other tool calls.
Verdict: FAIL #2 (major) — the answer's total does not match the tool result, and the extra $230 appears nowhere in the trace.
```

## Repetition

Because output varies, drive every branch that matters at least three times, in fresh sessions. Report the spread: three passes is a pass; one pass in three is a fail with the variance described; a flaky branch is a finding in its own right.

## The report

As in [FLOW-TREE.md](FLOW-TREE.md), with the transcript excerpt and the trace excerpt as the evidence for every finding, and the repetition results recorded per branch.

Done when: every turn in the tree has a rubric judgment written in sentences, every finding cites the answer and the trace, and the report contains no assertion that was checked by matching text.
