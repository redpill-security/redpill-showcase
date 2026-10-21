# RedPill — an AI red-teaming console that tells you what it did *not* test

RedPill attacks an AI model the way an adversary would — jailbreaks, prompt injection, system-prompt and data
leaks, harmful output — using several open-source attack engines at once, and turns their results into one
client-ready report that is explicit about what a result does and does not prove.

> **This repository is a showcase. It contains no source code.** RedPill is in a private beta.
> Reviewers can be given read access to the source repository on request (open an issue here).

---

## The problem

You have put an AI model, or an app around one, in front of users. Existing open-source red-team tools each
cover part of the risk, each speaks its own output format, and each can quietly report "nothing found" when it
actually failed to look — the attacker model refused to attack, the judge returned something unreadable, the
target answered every probe with an error. RedPill runs the engines together, normalises their results, and
treats "could not look" as its own outcome instead of a pass.

## What it does

1. **Point it at a model endpoint** (the target) — anything OpenAI-compatible (Ollama, vLLM, LM Studio,
   llama.cpp, OpenAI), and through promptfoo also Gemini, Anthropic, Bedrock and Azure. Optionally its host,
   for an infrastructure sweep.
2. **Choose a depth** (quick / standard / deep), threat categories and engines, confirm written authorisation,
   launch.
3. **The engines run:**
   - **garak** (NVIDIA) — a large library of published single-turn attacks, each with its own detector.
   - **PyRIT** (Microsoft) — objective sets and multi-turn attacks (e.g. crescendo) driven by a separate
     attacker model and scored by a judge model.
   - **promptfoo** — fires prompts from your own prompt bank, so it tests risks specific to your app.
   - **nuclei** — known-CVE and misconfiguration checks against the host.
4. **Everything is normalised** into one finding schema, graded, and mapped to the OWASP LLM Top 10 and
   MITRE ATLAS.
5. **You get** a console scan page with transcripts and evidence, a printable report (Executive and Technical
   files), and CSV / SARIF / JSON exports. A headless CLI does the same for CI pipelines.

## What makes it different

- **Proof outranks opinion.** RedPill can plant canary data in the target's context. If it comes back, or a
  forbidden tool is called, that is a recorded fact that overrides a "defended" verdict from a judge model —
  only ever toward a breach. The judge's own verdict is kept beside it.
- **Honest coverage.** A verdict RedPill cannot trust (for example one scored on an empty reply) is counted as
  *not evaluated*, never as *defended*. An engine that was selected but never ran counts against coverage.
  A report can read `INCOMPLETE` or `NOT EVALUATED` instead of reassuring you.
- **Pre-flight checks before every scan.** Before any attack is sent, RedPill checks that the attacker model
  actually produces attack content, that the judge grades known test cases correctly in both directions, and
  that the target answers — and records the result in the report.
- **No single score.** The headline is a sentence of counts: attacks that landed, out of how many reached a
  verdict, and how much of the plan was covered. A clean scan reads "No breach found", not "secure".
- **It measures itself.** RedPill keeps a blind, human-labelled answer key of its own past verdicts and
  measures its own false-clean rate with confidence intervals — under a written holdout rule so that answer
  key never quietly becomes training data.
- **Local-first.** Self-hosted in Docker, no telemetry; attacker and judge models can run on your own GPUs,
  so attack prompts and model replies never leave your network.
- **Safety rails.** A written-authorisation assertion at every launch, and a per-client list of authorised
  targets: launching at an unlisted host warns, and the report records it.

## Screenshots

| | |
|---|---|
| ![Scan setup](screenshots/01-setup.png) | ![A finished scan](screenshots/02-scan-detail.png) |
| *Scan setup: engines and what each will send at this depth* | *A finished scan: who attacked, who judged, coverage, provenance* |
| ![Evidence](screenshots/03-evidence.png) | ![Report](screenshots/04-report.png) |
| *Every finding with its evidence and verdict* | *The report summary: counts, coverage caveats, a second opinion* |

## Sample report

[`sample-report/`](sample-report/) holds the **Executive** file of a real scan against a lab model
(`gemma4:26b` on a local GPU, quick depth, all three attack engines) — never a client system. The Executive
file is the redacted hand-over copy: summaries and counts, without attack payloads or model replies.
Lab host addresses and workspace names were replaced with neutral placeholders (`lab-target.example`) in the
report and the screenshots; nothing else was edited.

## What it is not

- Not a guarantee that a model is safe. "No breach found" is a floor, not proof: these attacks, in this run,
  did not land.
- Not a penetration test of a whole application; it exercises the model and, optionally, its host.

## Status

Private beta, all rights reserved. Built on garak, PyRIT, promptfoo and nuclei, each under its own licence.
