# RedPill

### Test your AI the way an attacker would — and get a report that says plainly what was tested and what was not.

- **Several open-source engines, one report.** NVIDIA garak, Microsoft PyRIT, promptfoo and AgentDojo run
  together; their results are merged and graded in one place.
- **"Could not test" is never a pass.** If a test failed to run, the report says so instead of calling the
  model safe.
- **Ready to hand to a client or an auditor.** Executive and technical reports, mapped to OWASP and MITRE
  ATLAS, with CSV / SARIF / JSON exports.

It checks for jailbreaks, prompt injection, leaks of hidden instructions and data, harmful output, and AI
agents hijacked through the data they read.

**Private beta — [request access](https://github.com/redpill-security/redpill-showcase/issues/new?title=Beta%20access%20request).**

> This repository is a showcase. It contains no source code. Reviewers can be given read access to the
> source repository on request (see [Contact](#contact)).

## Who it's for

- **Before you launch an AI feature.** You built a chatbot, assistant or agent. Find out how it can be misused
  before your customers do.
- **Before you buy or connect a vendor's AI.** A supplier wants to plug their AI into your business. With their
  written permission, test it yourself instead of relying on their assurances.
- **When an auditor asks how you tested your AI.** Hand over a dated report mapped to the OWASP Top 10 for LLM
  and Agentic Applications and MITRE ATLAS, which says what was tested and what was not.

---

## The problem

You have put an AI model, or an app around one, in front of users. Existing open-source red-team tools each
cover part of the risk, each speaks its own output format, and each can quietly report "nothing found" when it
actually failed to look — the attacker model refused to attack, the judge returned something unreadable, the
target answered every probe with an error. RedPill runs the engines together, normalises their results, and
treats "could not look" as its own outcome instead of a pass.

## What it does

1. **Point it at your AI.** Works with self-hosted models and the major cloud AI services (some engines reach
   more of them than others). Optionally add the server it runs on, for a check for known security holes.
2. **Choose how deep to go** (quick, standard or deep) and what to test, confirm you have written permission, and
   start.
3. **Several engines test it at once:**
   - **garak** (NVIDIA) — a large library of published attacks.
   - **PyRIT** (Microsoft) — patient, multi-step conversations run by a separate attacking AI and graded by an
     independent judging AI.
   - **promptfoo** — your own test prompts, for risks specific to your business.
   - **AgentDojo** (ETH Zurich) — whether an AI agent can be taken over through what it reads (see below).
   - **nuclei** — known security holes and misconfigurations on the server.
4. **One set of results.** Everything is merged, graded and mapped to the OWASP Top 10 for LLM Applications and
   MITRE ATLAS.
5. **You get** a console with the evidence behind every finding, a report for leadership and one for engineers,
   and exports for your other tools. It can also run automatically as part of a software release.

## What makes it different

- **Hard proof beats opinion.** RedPill can hide a marker in the data the target can see. If that marker comes
  back, or the AI uses a tool it was told not to, the breach is recorded as fact, even if the judge model said the
  attack failed. Proof can only add a breach, never remove one, and the judge's verdict is kept beside it.
- **Honest coverage.** A result RedPill cannot trust, such as a verdict on an empty reply, is counted as
  *not evaluated*, never as *defended*. An engine that was chosen but never ran counts as a gap. A report can
  read `INCOMPLETE` or `NOT EVALUATED` rather than falsely reassure you.
- **Checks itself before every scan.** Before any attack is sent, RedPill makes sure the attacking model really
  attacks, the judge grades known test cases correctly, and the target answers. The result goes in the report.
- **Your data stays with you.** Runs in Docker on your own hardware, with no telemetry. The attacking and judging
  models can run on your own GPUs, so test prompts and model replies never leave your network.
- **Grades guardrails too.** Point it at a guardrail (Meta's Prompt Guard 2, LLM Guard, or a cloud filter) and it
  reports how many attacks were stopped and how many harmless prompts were wrongly blocked, each with a 95% range,
  so a small test never passes for a precise one.

## Testing AI agents that use tools

Many AI systems no longer just answer questions. They send email, pay bills and edit files. The risk shifts from
what the AI *says* to what it *does*.

RedPill gives the model an everyday job in a safe practice setting, with pretend email, calendar, banking and
files, and checks two outcomes:

- **Did it finish the job it was given?**
- **Did it also do something nobody asked for**, because of instructions hidden in an email or file it read?

Both answers come from what actually happened in the practice setting, not from what the AI says it did.

*What this does not cover:* it shows whether your model can be taken over through what it reads. It does not
test your own product's tools or connections.

## How it works

RedPill is built around one idea: a security tool must be as clear about what it did **not** check as about what it found. This page shows the path a scan takes and the rules that turn raw results into a verdict.

### How a scan runs

```mermaid
flowchart LR
    A["Target setup<br/>model endpoint, optional host"] --> B["Authorization check<br/>written sign-off at launch"]
    B --> C["Pre-flight checks<br/>attacker, judge and target"]
    C --> E1["promptfoo<br/>your own prompt bank"]
    C --> E2["garak<br/>published attack library"]
    C --> E3["Microsoft PyRIT<br/>multi-turn attacks"]
    C --> E4["AgentDojo<br/>agent hijack tests"]
    C --> E5["Infrastructure sweep<br/>known CVEs and misconfigurations"]
    E1 --> J["Independent judge model<br/>grades each reply"]
    E2 --> J
    E3 --> J
    E4 --> J
    E5 --> N["Normalised findings"]
    J --> P["Proof checks<br/>outrank the judge"]
    P --> N
    N --> R["Report<br/>HTML tiers, SARIF, CSV"]
```

You point RedPill at a model endpoint, choose a depth and the engines, and confirm written authorisation. Before any attack is sent, it checks that the attacker model really attacks, that the judge grades known test cases correctly, and that the target answers. The engines then run together, and an independent judge model reads the replies. Proof checks come last and can overrule the judge, but only toward a breach. Everything lands in one report, with HTML for three audiences (Executive, Technical, Full), plus SARIF, CSV and JSON exports.

### How RedPill decides

#### (a) Breach, No breach found, or Not evaluated

```mermaid
flowchart TD
    A["One attack result"] --> B{"Did we get a<br/>readable answer?"}
    B -- "no: empty, cut off, or never graded" --> NE["NOT EVALUATED<br/>counted in its own column"]
    B -- yes --> C{"Did the attack land?"}
    C -- yes --> BR["BREACH"]
    C -- no --> D{"Could the detector<br/>actually tell?"}
    D -- "no: it cannot answer this question" --> NE
    D -- yes --> OK["No breach found"]
```

Every result ends in one of three places. **Not evaluated** means "we did not learn whether the attack landed", for example because the target replied with nothing, the answer was cut off, or the detector was the wrong tool for the question. Those results leave every rate and count, and they are reported in their own column. They are never counted as safe, because a missing answer is not evidence of a defence. If a selected engine never ran at all, that counts against coverage too, and the report can read `INCOMPLETE` instead of reassuring you.

#### (b) Severity, and when proof outranks the judge

```mermaid
flowchart TD
    A["Judge's verdict"] --> B{"Planted data came back,<br/>or a forbidden tool was called?"}
    B -- yes --> C["BREACH<br/>judge's verdict kept beside it"]
    B -- no --> D{"Judge says breach?"}
    D -- no --> E["Not a breach<br/>severity: info"]
    D -- yes --> F["Start from the category's impact"]
    F --> G["Nudge at most one step<br/>for how easy the attack was<br/>and how reliably it works"]
    G --> H["Severity: low to critical"]
    C --> F
```

RedPill can hide a marker in the data the target can see. If that marker comes back, or the AI uses a tool it was told not to, that is a recorded fact, not an opinion, so the finding becomes a breach whatever the judge said. The judge's own verdict is kept and shown beside it. No rule ever turns a breach into "defended", so a bug cannot hide a leak. Severity then says how bad a breach is, based on the kind of risk and how easy and reliable the attack was, and the report says when a test was too small to be sure.

### What the report tells you

- **What was tested:** engines, depth, threat categories, attacker and judge models, and how much of the plan was covered.
- **What was NOT tested, and why:** results marked not evaluated, engines that did not run, and OWASP classes that were out of scope, each with a plain-English reason.
- **Evidence for every finding:** transcripts, the verdict, which detector or judge decided it, and any proof that overruled it.
- **No single score:** a headline sentence of counts (attacks that landed, out of how many reached a verdict) and a clean scan reads "No breach found", never "secure".
- **Framework mapping:** findings are mapped to the OWASP LLM Top 10 and MITRE ATLAS, with a row for every OWASP class marked evaluated, adjacent, not run or out of scope. An OWASP Agentic Applications appendix is included, and it is explicit about which risks the scan did not test.
- **Who looked:** how many breaches a person has checked, and how many different prompts stand behind a count.

"No breach found" is a floor, not proof: these attacks, in this run, did not land.

## Reporting and compliance

- **Three report tiers from one scan:** an Executive summary for leadership, a Technical file for engineers, and a
  Full file with every transcript.
- **Mapped to the frameworks your auditors use:** the OWASP Top 10 for LLM Applications, the OWASP Top 10 for
  Agentic Applications and MITRE ATLAS, with a row for every risk saying whether it was tested, and why not if not.
- **Remediation guidance:** each breached MITRE ATLAS technique lists MITRE's own published mitigations, so the
  fix list starts from a recognised source.
- **Healthcare mapping:** the practice clinic app maps its results to the HIPAA rules they touch.
- **Machine-readable exports:** SARIF for code-scanning dashboards, plus CSV and JSON for your own tooling.
- **Evidence you can hand over:** every finding carries its transcript, its verdict and who or what decided it.

## Screenshots

| | |
|---|---|
| ![Scan setup](screenshots/01-setup.png) | ![A finished scan](screenshots/02-scan-detail.png) |
| *Scan setup: engines and what each will send at this depth* | *A finished scan: who attacked, who judged, coverage, provenance* |
| ![Evidence](screenshots/03-evidence.png) | ![Report](screenshots/04-report.png) |
| *Every finding with its evidence and verdict* | *The report summary: counts, coverage caveats, a second opinion* |

## Sample report

[`sample-report/`](sample-report/) holds the **Executive** file of a real scan against a lab model
(`gemma4:26b` on a local GPU, quick depth, garak + PyRIT + promptfoo; captured 2026-10-21, before AgentDojo joined the default set) — never a client system. The Executive
file is the redacted hand-over copy: summaries and counts, without attack payloads or model replies.
Lab host addresses and workspace names were replaced with neutral placeholders (`lab-target.example`) in the
report and the screenshots; nothing else was edited.

## More details

- **No single score.** The headline is plain counts: how many attacks got through, out of how many could be
  judged, and how much of the plan was covered. A clean scan reads "No breach found", never "secure".
- **It checks its own accuracy.** People review samples of RedPill's past results without seeing its verdict
  first, and RedPill measures how often it wrongly called something safe, with an honest margin of error. Those
  reviewed cases are never used to tune RedPill, so the accuracy figure stays honest.
- **It double-checks a clean result.** Before trusting a clean agent test, RedPill runs a known takeover through
  its own scoring. If that would not have been caught, the scan is marked untrustworthy, not clean.
- **Says who looked.** The report's first page says how many breaches a person has checked, how many truly
  different attacks stand behind a count, and what a test limit left out. Exporting a report with unchecked
  breaches shows a warning first.
- **Safety rails.** Written permission must be confirmed at every launch, and each client has a list of approved
  targets. Starting a scan against anything else triggers a warning, and the report records it.

## Coming next (not built yet)

- A practice **bank** app (card data, PCI DSS mapping), with the guardrail tested with and without. (The practice
  **clinic**, **receptionist**, **shared-memory** and **coding-assistant** apps are built and in beta.)
- Finding **MCP tool servers** and **unapproved AI services** on networks you authorise, read-only.

## What it is not

- Not a guarantee that a model is safe. "No breach found" is a floor, not proof: these attacks, in this run,
  did not land.
- Not a penetration test of a whole application; it exercises the model and, optionally, its host.

## Status

Private beta, all rights reserved. Built on garak, PyRIT, promptfoo, AgentDojo and nuclei, each under its own licence.

## Contact

To request beta access, or reviewer access to the source, [open a request here](https://github.com/redpill-security/redpill-showcase/issues/new?title=Beta%20access%20request) (a free GitHub account is
needed). Say who you are and what you would like to test; requests are answered by the RedPill team.
