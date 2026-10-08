---
name: exploratory-testing-techniques
description: >-
  Use when planning, running, or reporting an exploratory test session and you
  need heuristics, oracles, tours, attacks, or data values to pick a charter's
  lenses, find a fresh angle when stuck, argue why something is a bug, or
  write the MCOASTER / BUG-NN report.
---
# Exploratory testing techniques

> **Credit:** Adapted with credit from **Dan Ashby**'s *Exploratory Testing Skill*, https://github.com/danashby/Exploratory-Testing-Skill (adapted with credit; original README states MIT, the repo has no LICENSE file). Heuristic authors are credited as Ashby credits them: FEW HICCUPPS (Michael Bolton & James Bach), SFDIPOT and CRUSSPIC STMPL (James Bach), FCC CUTS VIDS tours and MCOASTER (Michael D. Kelly), testing attacks and RIMGEA (Cem Kaner), data heuristics from the Test Heuristics Cheat Sheet (Elisabeth Hendrickson, James Lyndsay, Dale Emery), SBTM (James & Jon Bach). Full list and authors: `references/heuristics.md`.

The point of QA is to break things. This is the technique menu; the procedure for one session (charter, gate, recording, report delivery) is the `qa-test-session` skill, and how to judge what you see is `qa-judge-like-a-real-user`. The tester finds and reports; it never fixes code.

## When to use

- Writing a charter: choosing which lenses to point at a target.
- Mid-session: stuck, or needing a fresh angle ("what haven't I varied?").
- Arguing why an observation is a bug (name the oracle).
- Writing the session report.

## Charter format (SBTM, James & Jon Bach)

> **"Explore [target], with [resources/data/tools], to discover [information about risks]"**

Keep charters to roughly 60–120 minutes of work. Heuristics are a menu of lenses, not a checklist: select what is relevant and combine freely.

## Pick heuristics for a charter

Pick **one or two from each row** that fit the target and risk. All are in `references/heuristics.md`.

| Need | Reach for |
|---|---|
| Recognise a problem (oracles) | FEW HICCUPPS: always on; cite the letter in every bug |
| Decide what to cover | SFDIPOT (product areas); CRUSSPIC STMPL (quality attributes); RCRCRC (after a change); W5HE / MIDTESTD (before starting) |
| Move through the app | FCC CUTS VIDS tours: Feature, Complexity, Claims, Configuration, User, Testability, Scenario, Variability, Interoperability, Data, Structure |
| Break it | Kaner attacks: Long Name, Special Characters, Boundary Value, Invalid Type, Null/Empty, Overload, Interruption, Sequence; plus the Part 5 data-type attack cheat sheet (string, numeric, date/time, file/path) |
| Choose values | Goldilocks, ZOM, CRUD (+ Dependencies), BME, Boundaries, Some/None/All, Sequences, Count, Sorting, Input Method Variations, Configurations, State Analysis, Map Making |
| Stress / resilience | Multi-User/Concurrency, Interruptions, Starvation, Flood; FIBLOTS/IVECTRAS for performance |
| Errors seen | FAILURE (Ben Simo) to judge the error handling itself |
| Mobile and small screens | I SLICED UP FUN; the size sweep in `qa-judge-like-a-real-user` |
| Prioritise order | SLIME, DUFFSSCRA; "Bugs Cluster": found one, dig nearby |
| Quick sweeps | Part 13 rapid questions: security, performance, accessibility, i18n, concurrency & state |

In a headless browser, many of these map straight onto Playwright: Interruptions and Starvation via `context.setOffline(true)` or route throttling and aborts (`page.route`), Multi-User/Concurrency via two contexts side by side, Variability via viewport, locale (`locale`, `timezoneId`), and colour-scheme emulation, Input Method Variations via `keyboard.type` versus `fill` versus clipboard paste versus touch.

All attacks are in scope; the risky ones need the manager's okay first (see the risky-probe gate in `qa-test-session`).

## Report formats

Produce the report automatically when the session ends; don't wait to be asked.

**Tell the story, don't just summarise.** The centrepiece is a chronological session narrative: what you did, what you expected, what you actually saw (exact text, timings, visual state, console and network errors), what you tried next and why, plus hunches, small weirdnesses, slow or confusing moments, and surprises that weren't judged bugs. Don't pre-filter: when in doubt, write it down, because the reader may spot an oddity you didn't. Keep timestamped notes as you go so the story isn't rebuilt from memory.

Structure, in prose rather than bullet fragments: a **MCOASTER** header block (Michael D. Kelly: Mission, Coverage, Obstacles, Audience, Status, Techniques, Environment, Risks), then **1. The brief**, **2. Tools used**, **3. What I did** (the narrative, with times and evidence paths inline), **4. What I didn't do or couldn't do** (including risky probes not approved, and coverage against the charter), **5. What happened** (results, the BUG-NN index pointing back into section 3, a summary table of ID | criterion | severity | screen | status, and recommendations grouped Priority 1 / 2 / 3 for the owner to triage, advisory only), **6. What was interesting**, and **7. What we learned** (including suggested next charters). Fill-in templates are in `qa-test-session/references/templates.md`.

BUG-NN entry:

```
BUG-NN: <one-line title>
- Criterion: <oracle or standard, e.g. FEW HICCUPPS "Claims", WCAG 2.2 AA 1.4.3>
- Severity: CRITICAL / HIGH / MEDIUM / LOW / question: intended?
- Screen / area: <page, component, URL, sizes>
- Repro steps: 1. … 2. … 3. …  (exact inputs; minimal conditions per RIMGEA)
- Expected vs actual: <what should happen> / <what happened, exact text>
- Evidence: shots/NN-<slug>.png; video/<file>.webm @ mm:ss; traces/<name>.zip; narrative @ HH:MM
- Expected behaviour / likely area (advisory, the tester does not fix): <…>
```

Write bugs with RIMGEA (Cem Kaner): Replicate, Isolate, Maximise, Generalise, Externally describe, And report. Any memory or resource leak is CRITICAL.

## Reference

- `references/heuristics.md`: every oracle, coverage model, tour, attack, data heuristic, data-type attack value, regression, error-handling and reporting heuristic, wisdom rule, SBTM, rapid questions, and the mnemonic index, with authors.
