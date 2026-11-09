# Lecture 02 Answers

## Exercises

## Exercise

- Model and settings: Qwen/Qwen3.5-0.8B, LoRA (no quantization), r=16, alpha=32, lr=2e-4, 20 epochs, warmup_steps=6, seed 42, greedy decoding (do_sample: false)
- I ran the training myself (executed_training_myself: true)
- Training: 40 steps, 221 s, final loss 0.0137 (1.56 at step 10)
- format_rate: 0.095 → 1.000
- content_rate: 0.000 → 0.905 (19/21)
- Regressed questions: none
- Remaining failures (2), both unfounded assertions on questions not in the FAQ:
  - 「成績はいつ出ますか」: answered with the deadline text, needs_review=false
  - 「オフィスアワーは事前予約が必要ですか」: answered with the office hours text, needs_review=false
- Knowledge questions (before → after):
  - Final assignment deadline: format_error → asserted
  - Instructor's name: format_error → needs_review
  - Next lecture topic: format_error → needs_review
  - Control (assignment deadline): format_error → asserted
- What training changed: the output format (valid JSON with both keys, 100% compliance) and the wording, which now follows the FAQ text.
- What it didn't guarantee: judgment on unknowns. For some unseen questions the model matched the closest FAQ item and answered confidently instead of asking for review.

## Homework

**Level:** Basic

**Changed:** num_train_epochs 20 → 5. Everything else was kept the same (model, lora_r=16, lr=2e-4, seed 42). I restarted the session and ran from the top.

| | 5 epochs | 20 epochs (original) |
|---|---|---|
| Steps | 10 | 40 |
| Final loss | 1.571 (step 10) | 0.0137 (step 40) |
| Training time | 59 s | 221 s |
| format_rate | 1.000 | 1.000 |
| content_rate | 0.714 (15/21) | 0.905 (19/21) |
| Regressions | none | none |

**Observations**
1. Format is learned quickly: both runs reach 100% format compliance, so more epochs did not help the format.
2. More epochs improved content from 71% to 90%. At 5 epochs the loss was still about 1.57 at step 10 (similar to the 20-epoch run at step 10), so that run was very short and had barely started to converge.
3. The failure types differ. At 5 epochs (6 failures): 2 over-refusals on grounded questions ("not in the FAQ" for something that is), 3 ungrounded questions with needs_review=true but wrong or missing content, and 1 unfounded assertion. At 20 epochs (2 failures): both are unfounded assertions on ungrounded questions. So accuracy improved, but the remaining errors are the unsafe type.
4. Knowledge questions gave the same categories in both runs.
5. Caveats: only 21 test questions (one question is about 4.8 points), one seed, and GPU training is not fully reproducible, so the 5 vs 20 epoch gap is suggestive, not conclusive.

**Extra (Section 8, 4-bit quantization, no training):** weight memory dropped from 3.01 GB to 0.76 GB. Without any training, format_rate was 0.667 and content_rate 0.333 (vs 0.095 / 0.000 in 32-bit). However, it asserted "なし" or "はい" with needs_review=false for 4 of the 6 ungrounded questions, and one output had needs_review as the string "false" (a type error). Quantization changes outputs with no consistent direction, so before/after comparisons must use the same quantization.

**When fine-tuning fits and when it doesn't**
- It fits when the goal is a fixed output format or style and the training data shows the behavior you want.
- It does not reliably give "say you don't know when unsupported". With 14 examples, that judgment did not transfer, and more epochs pushed the model toward matching unseen questions to memorized FAQ answers.
- For facts that change (deadlines, FAQs), fine-tuning bakes in content that can go stale. Searching the source (retrieval) and using training only for the format is safer.

**What I verified:** I ran both trainings in Colab myself and checked the numbers against the Colab output and the result JSON files.
