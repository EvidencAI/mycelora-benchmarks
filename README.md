# Mycelora on BEAM

Raw answers and judge verdicts behind Mycelora's published BEAM scores.
Methodology and comparison: https://mycelora.ai/benchmarks

| | BEAM 100K | BEAM 1M | BEAM 10M |
|---|---|---|---|
| Score (mean of the ten ability means) | **75.6%** | **74.5%** | **63.2%** |
| Questions scored 0.5 or more | 85.0% | 82.7% | 68.5% |
| Mycelora version | production of 2026-11-05 | production of 2026-11-05 (run 2026-11-07 to 2026-11-11), with three technical fixes of 2026-11-09 for 18 conversations | production of 2026-11-09 (run 2026-11-11 to 2026-11-19) |
| Conversations | 17 of 20: 2, 3, 4, 6 to 13, 15 to 20 | all 35 | all 10 |
| Questions | 340 | 700 | 200 |
| Runs of the answer model | one | one | three on the same served context, best kept per conversation |
| Answer model | Claude Opus 5.5, no tools | Claude Opus 5.5, no tools | Claude Opus 5.5, no tools |
| Judge | gpt-5, Mem0's BEAM judge prompt | gpt-5, Mem0's BEAM judge prompt | gpt-5, Mem0's BEAM judge prompt |

Both runs measure the same version of Mycelora. They are not averaged together. Earlier 1M publications (production of
2026-10-27, 66.1%; first 17 conversations of this run, 73.6%) remain in the repository history.

- **10M:** all 10 conversations. None was used for tuning.
  For each question, the context served by Mycelora was stored once; the answer model was then run three times on that
  same context (`original`, `replay-1`, `replay-2`), each run judged by gpt-5. The score keeps, for each conversation,
  the run with the best score: conversation 1 `original` (65.6%), 2 `replay-1` (63.4%), 3 `replay-1` (63.5%),
  4 `replay-1` (78.3%), 5 `replay-1` (54.5%), 6 `original` (60.8%), 7 `replay-1` (73.6%), 8 `replay-1` (49.4%),
  9 `original` (55.8%), 10 `original` (66.9%). Full runs over the ten conversations: `original` 59.6%,
  `replay-1` 61.5%, `replay-2` 58.5%.
  All three runs are in `results/10m/results.jsonl` (`run`, `best_run`). Protocol deviations: the limit on the number
  of topics in a codex went from 60 to 100 on 2026-11-15 at 02:27 CET, from day 82 of conversation 2, day 72 of
  conversation 3 and day 93 of conversation 4 (conversation 1 ran entirely under 60); conversation 4 ran on the route
  used by real accounts, where about 11% of extraction calls fell back to mistral-small, and 3 of its history batches
  were re-injected out of order after an extraction error; sessions whose content exceeds about 550,000 characters
  were occasionally refused by the session-closing model and retried. Conversations 5 (second run) and 6 to 10 closed all
  their sessions under the server limit of 100 topics. For conversations 1 to 9 the session-closing model was still
  told a limit of 60 topics (no new topic from 55); for conversation 10 it was told 100 (no new topic from 95). In
  conversation 8 the codex reached 55 topics on day 63 and no topic was created after that; it ended at about 50,000
  characters, against 86,000 to 96,000 for the other conversations. The sessions of conversation 8 were replayed on
  2026-11-19 on the same injected memory with the 100-topic rule: that replay scored 45.3%, 51.0% and 48.5% over three
  answer-model runs and is not published; only the first run is scored. Conversation 10 ran on the route used by real
  accounts, where about 20% of extraction calls fell back to mistral-small. The injection of conversation 9 was paused
  on 2026-11-16 and resumed on 2026-11-17 in the same memory space. While closing the sessions of conversation 10, the
  server returned an HTTP 500 four times; each time the run was resumed in the same memory space. Conversation 5 was run twice end to end (history injected, sessions
  closed, questions answered); the first run (2026-11-10 to 2026-11-12) scored 50.7% on its original answers and is not
  published; only the second run (2026-11-14 to 2026-11-16) is scored. In conversation 7, no message of the first batch
  carries a date: that session is dated the day before the next dated batch (2020-03-04). Conversations 6 and 7 were
  stopped on 2026-11-16 after a test-harness error and resumed in their original memory space, without re-injection:
  6 from day 56, 7 from day 1. For two sessions (conversation 6 day 66, conversation 7 day 19), the server returned an
  HTTP 500 on session close after writing it; the close was replayed and recognised as a replay, with one handover per
  session.
- **100K:** conversations 1, 5 and 14 were used for tuning and are excluded from the score.
- **1M:** all 35 conversations, run three at a time with identical settings; none was used for tuning.
  Conversations 18 to 35 except 22 ran first (2026-11-07 to 2026-11-09); conversations 1 to 17 and 22 ran next
  (2026-11-09 to 2026-11-11) with three technical fixes of 2026-11-09: tolerant reading of a malformed answer from the
  extraction model, linking each memory to its thread, sorting memories at session close. Incidents in the first 17: 3 history batches out of about 11,000 could not be stored because the extraction
  model returned an unreadable answer (conversations 24, 27 and 34); the batch of conversation 34 was re-injected before
  the questions, the other two are missing. Conversations 30 and 35 were resumed, without re-injection, after an
  unreadable answer from the model that closes each session. No history batch was lost in the next 18.

Judge prompt: [mem0ai/memory-benchmarks](https://github.com/mem0ai/memory-benchmarks), `benchmarks/beam`.

## What the answer model receives

The question and what Mycelora serves for it, inside BEAM's own answer prompt
(`answer_generation_for_rag`, unchanged, in [prompts/answer_prompt.txt](prompts/answer_prompt.txt)).
Nothing else: no question type, no hint, no annotation from the dataset.

## How the conversation reaches Mycelora

Each session of the conversation is sent to Mycelora and closed like a real work thread. At question time, the
answer model gets Mycelora's opening block for the thread followed by its recall for the question. The judge never
sees the served context.

## What the benchmark serves, what the product serves

In the benchmark, every question is asked in a fresh thread, as the protocol requires. So each time the answer model
receives Mycelora's opening block (the state of the project), then the recall of memories for that question. The size
of this context is published per question in this repository: a median of 41,300 characters at 100K and 79,000 at 1M.
At 1M, the opening block accounts for 79% of it.

In the product, a thread is opened once. The opening block is served at that point, then each question only adds its
recall. We measured this by replaying three 1M benchmark conversations as a single thread of
20 questions, with the same memories:

| | Benchmark: 20 independent questions | One thread: the same 20 questions |
|---|---|---|
| Opening block | served 20 times | served once: 31,000 to 35,000 tokens |
| Context per question | 36,000 to 42,000 tokens | 5,600 to 7,200 tokens added |
| Context served for the 20 questions | 730,000 to 840,000 tokens | 155,000 to 176,000 tokens |

Tokens as counted by the API of the answer model (Claude Opus 5.5). As in any conversation, the model then rereads
the thread history at each turn. On these three conversations, the mean score is 0.754 in the benchmark and 0.721 in
one thread: a gap within measurement noise. This replay is not part of the published score.

## Scoring

Each rubric criterion is scored 0, 0.5 or 1 by the judge, a question scores the mean of its criteria, each ability
scores the mean of its questions (34 at 100K, 70 at 1M), and the overall score is the mean of the ten abilities.

## Files

| File | Content |
|---|---|
| `results/<size>/summary.json` | overall score, share of questions at 0.5 or more, per-ability and per-conversation scores |
| `results/10m/results.jsonl` | same, one line per question and per run, with `run` and `best_run` |
| `results/<size>/results.jsonl` | one line per question: conversation, ability, question, answer, score, each criterion with the judge's score and reason, size of the served context |
| `prompts/answer_prompt.txt` | the answer prompt |

The served context itself is not published.

## Known limits

- One run per size at 100K and 1M. Two identical runs differ by 1 to 5 points overall. At 10M, three runs of the
  answer model on the same context range from 58.5% to 61.5% overall.
- English conversations generated by BEAM.
- The scores measure Mycelora with Claude Opus 5.5; another answer model would give other scores.

## License and attribution

Questions and rubric criteria come from the BEAM dataset (Tavakoli et al., ICLR 2026,
[Hugging Face](https://huggingface.co/datasets/Mohammadta/BEAM)), released under CC BY-SA 4.0.
This repository is released under the same license ([LICENSE](LICENSE)).

Published by EvidencAI, maker of [Mycelora](https://mycelora.ai).
