# RuleForge: agentic rule generation for CloudGuard Labs

HashedIn by Deloitte · Campus PM Hiring 2026 · Case 14
Alex Shajan · IIM Shillong · 2025PGP280

> AI drafts the evidence. SMEs decide. Every finding stays provable.

## What's in this repo

| File | What it is |
|---|---|
| `Case14_Deck.pptx` | The submission deck (23 slides) |
| `RuleForge_Wireframes.html` | Clickable prototype. Open it in a browser: Detect → Draft & verify → Publish → Monitor |
| `Case14_Golden_Dataset.xlsx` | The golden dataset used to design and evaluate the solution |

## The solution in one paragraph

CloudGuard's 6 SMEs take 3–5 SME-days per rule, and AWS alone produced 11 rule-worthy changes in 13 days. That is 33–55 SME-days of work against the 54 SME-days the whole team has in that period. RuleForge is a fixed workflow of five agents: Watcher, Research, Author, a deterministic Test harness, and Critic. Together they turn each cloud change into an evidence pack: cited sources, a drafted rule, sandbox test results and edge-case flags. An SME approves, edits or rejects it, giving a reason code when rejecting. Rules then go through shadow → canary → GA behind a kill switch. The MVP is AWS only, with 100% SME sign-off.

## The golden dataset

All the data is real and public. Nothing is synthetic.

- **CIS AWS Foundations Benchmark v7.0.0 and v6.0.0:** recommendation IDs, titles, sections, profile level and assessment type, as mapped in Prowler.
- **Prowler (Apache-2.0):** check metadata (severity, risk, remediation, Terraform/CLI fixes) and PASS/FAIL messages taken from the check source code.
- **AWS What's New RSS:** the 100 most recent announcements, 14–26 Oct 2026.

### Sheets and the agent each one tests

| Sheet | Rows | Tests | What "correct" means |
|---|---|---|---|
| `Positive_Set` | 86 | Research + Author agents, Test harness | Right check logic, severity, PASS/FAIL message and citation; compliant fixture passes, non-compliant fails |
| `Edge_Cases` | 51 | Critic agent | Avoids known traps, such as controls CIS marks manual but that are partly automatable (32), one recommendation → many rules (9), no automated check exists (4, where it must not invent a rule) and account-level overrides (E-050) |
| `Triage_Set` | 100 | Watcher agent | Signal vs noise: 11 need a new or updated rule, 89 don't |
| `Version_Drift` | 31 | Drift remapping (MVP) / Maintenance agent (V1) | Correct action per CIS v6.0 → v7.0 change: renumber, re-map, add or retire |
| `Eval_Scorecard` | — | Go/no-go gate | Formulas compute rule, severity and citation accuracy, average SME review minutes, and triage precision and recall |

`Proposed_Risk_Tier` sets the sign-off policy: **T3** = critical severity, or high severity in IAM or network, and needs 2 SMEs. **T2** = other high or medium. **T1** = low. 26 of the 86 rows are T3.

### Where it is used

**1. Before launch: backtest (pilot weeks 1–3).** Hide the reference rules, regenerate them from the source material, and score with `Eval_Scorecard`. The KPI floors are recalibrated from this run.

**2. In CI: regression gate.** The full set runs on every model, prompt or rule-format change, at every promotion (dev → staging shadow → prod canary → GA), and in a weekly regression run.

**3. At go/no-go: release gate.** These are proposed floors, recalibrated after the backtest:

| Metric | Floor |
|---|---|
| Citation accuracy | 100% (hard gate) |
| Rule accuracy (all gates passed) | ≥ 80% |
| Severity accuracy | ≥ 90% |
| Triage recall (needs rule) | ≥ 95% |
| SME minutes per rule | ≤ 120 |

**4. In the deck**

| Slide | Use |
|---|---|
| 5 | Capacity maths from Triage_Set (11 rule-worthy items in 13 days) |
| 9 | Eval gate in the architecture |
| 13 | MVP drift-remap scope from Version_Drift |
| 14 | Backtest in pilot weeks 1–3 |
| 15 | KPI definitions match Eval_Scorecard |
| 16 | CI gate at every promotion |
| 18 | Demo rule built from P-036, P-037 and E-050 |
| 20 | Dataset overview and known limits |
| 21–22 | Risk checks and go/no-go floors |

**5. In the prototype**
- **Detect:** the 100 / 11 / 89 counts and the inbox items come from Triage_Set. The "31 changes" tile comes from Version_Drift.
- **Draft & verify:**
  - The rule, the four fixtures and the CLI fix come from P-036.
  - The Critic's flag is E-050.
  - The T2 badge comes from Proposed_Risk_Tier.
- **Publish:** the finding text, the risk text and the CLI/Terraform fix come from P-036.
- **Monitor:** the sheet counts and the release-gate metrics mirror Eval_Scorecard.

### Known limits

- **Ground truth is Prowler's reading of CIS.** For example, CIS 3.1.4 is checked on two of the four Block Public Access settings (IgnorePublicAcls, RestrictPublicBuckets).
- **Triage labels are drafts.** `Label_Status` stays "verify" until an SME has reviewed each row.
- **11 Positive_Set rows have no Terraform fixture.** Examples are root MFA and org-level controls. They need standing accounts in the sandbox AWS Organization.
- **The scorecard is a template.** The figures above are targets, not measured results.

## Sources

- [Prowler CIS AWS compliance mappings and checks](https://github.com/prowler-cloud/prowler) (Apache-2.0)
- [CIS Amazon Web Services Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [AWS What's New RSS feed](https://aws.amazon.com/about-aws/whats-new/recent/feed/)

CIS Benchmark text is © Center for Internet Security. Only recommendation IDs and titles are included here.
