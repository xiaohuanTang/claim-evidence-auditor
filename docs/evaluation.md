# Evaluation protocol

## Scope

This is a **20-claim constructed evaluation** for directional product validation. It is not a benchmark and does not claim statistical significance.

The goal is to compare three workflow versions on the same claims and the same model:

| Version | Purpose |
| --- | --- |
| v0 | Naive RAG baseline |
| v1 | Add deterministic pre/post guardrails |
| v1.1 | Keep the structure; improve the evidence rubric |

## Dataset

- 20 short academic claims.
- 10 `Supported`, 5 `Weak`, 5 `Unsupported`.
- 2 claims contain fabricated citation numbers outside the paper's reference range (`[12]`, `[13]`; the paper has 11 references).
- The full paper is not published. `data/claims.json` contains only the short claims and human labels.

## Label definitions

| Label | Definition |
| --- | --- |
| `Supported` | The paper provides checkable evidence: a table, experiment, or citation that supports the claim. |
| `Weak` | The paper has a related statement, but evidence is indirect, qualitative, incomplete, inconsistent, or only in an attachment. |
| `Unsupported` | No basis, a contradiction, a fabricated citation, a wrong attribution, or an author hypothesis presented as a conclusion. |
| `Hypothesis` (sub-tag) | The claim is an author's design assumption or causal inference, not an external citation or experimental result. |

## Versions

### v0 — naive RAG

`Start → Knowledge Retrieval → LLM → End`

No citation parsing, no precheck, no JSON/label guard. The LLM output goes directly to the result.

### v1 — guardrails

`Start → Code A → IF → (precheck) Code B → Aggregate → End`

`IF false → Knowledge Retrieval → LLM → Code C → Aggregate → End`

- **Code A** parses citation numbers and flags any `n < 1` or `n > 11`.
- **Code B** returns a fixed `Unsupported` result with `fake_citation=true` for invalid citations. No retrieval or LLM call.
- **Code C** parses the LLM output, normalizes the label, truncates evidence, enforces the output schema, and marks `ERROR` when parsing fails.

### v1.1 — evidence rubric

Same workflow as v1. Only three changes:

1. The evidence-grading prompt distinguishes **"the paper says it"** from **"the paper provides checkable evidence."**
2. Code C strips DeepSeek thinking blocks before JSON parsing. The tag is written here as a code block so it is not confused with HTML:

```text
<think>...</think>
```

3. `max_tokens` increases from 1024 to 2048.

## Metrics

| Metric | Definition |
| --- | --- |
| Three-level accuracy | Correct label / 20 |
| Valid-output accuracy | Correct label / outputs that parsed successfully |
| Format error rate | `ERROR` label or failed run / 20 |
| Fake-citation detection | `fake_citation=true` for the 2 constructed cases |
| Weak-risk recall | #11–#15 labeled `Weak` or `Unsupported` / 5 |
| Strict Weak hit | #11–#15 labeled exactly `Weak` / 5 |
| Hypothesis detection | #16–#17 with `hypothesis=true` / 2 |
| Token cost | `total_tokens` from the workflow run, averaged over 20 |
| Latency | workflow `elapsed_time`, averaged over 20 |

## Controlled variables

- Model: DeepSeek `deepseek-flash`
- Temperature: 0
- Retrieval: Dify knowledge base, economy keyword index, `top_k=10`
- Dataset: same 20 claims for all versions
- Prompt: identical for v0 and v1; only v1.1 changes the evidence rubric
- `claim_type` is logged by Code A but **not passed to the LLM** in the main comparison, to keep the v0/v1 comparison clean.

## Known limitations

- Small constructed sample; no confidence intervals or significance claims.
- Keyword retrieval, not vector retrieval.
- Reference metadata is entry-level (author/title/venue/year), not full-text PDFs.
- The two fabricated citations were caught by v0 in this run too; do not generalize that v0 is reliable for fake-citation detection.
- Remaining v1.1 errors are rubric-boundary cases, not retrieval failures.
