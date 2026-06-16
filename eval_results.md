# Eval Results

## Task 2 — LLM-as-judge pass-rate table

Two variants scored against the same fixed test set with a judge LLM and an
explicit rubric.

| Variant | Cases | Passed | Pass rate |
|---------|-------|--------|-----------|
| Variant A (few-shot)        | 12 | 0 | 0%  |
| Variant B (embeddings + NN) | 12 | 1 | 8%  |

**Rubric used by the judge:**

> You are a strict grader for a support-ticket classifier.
>
> Ticket: "{question}"
> Correct label: {expected}
> Model's predicted label: {answer}
>
> Rubric: PASS if the predicted label exactly matches the correct label.
> FAIL otherwise. Respond with ONLY one word: PASS or FAIL.

**Verdict (2–3 sentences):**

> Neither variant "won" on the judge's pass-rate — but the judge's numbers are not trustworthy here. Manually comparing `true_label` to each prediction, few-shot actually got 10/12 correct and embeddings got 2/12 correct, yet the judge marked 11 of those 12 correct few-shot predictions as FAIL, including exact string matches like predicted `billing` vs correct `billing`. This means the judge is not reliably applying its own rubric — it appears to be defaulting to FAIL regardless of whether the labels match, rather than doing the literal string comparison it was instructed to do.
>
> I do **not** trust this judge in its current form. A judge that fails on trivial exact-match cases (the easiest possible rubric to apply) is unsafe to use for picking a winning variant; relying on its pass rate here would have led to the wrong conclusion (it makes both variants look equally bad, and even ranks embeddings — the actually weaker classifier — above few-shot). Before trusting it, I'd want to inspect the judge's raw output text (not just the parsed PASS/FAIL) to see whether `llama3.2:3b` is ignoring the instruction, getting confused by the prompt format, or the parsing logic (`verdict.startswith("PASS")`) is silently mismatching valid responses.

**A case where the judge looked wrong:**

> Ticket: "How long does it usually take for a refund to show up on my card?" — correct label `billing`, few-shot predicted `billing` (an exact match), yet the judge scored this **FAIL**. Since the predicted label is identical to the correct label, this is the clearest possible case of judge error: there is no ambiguity in the rubric for the judge to misapply, which points to the judge model itself failing to follow its own "PASS if exact match" instruction rather than any genuine disagreement about ticket classification.