# QA skill pack (Cursor)

> **Status: not fully reviewed or endorsed.** This is the QA skill pack I'm currently running with my QA bot, shared as-is. It's a working snapshot and still changing, so treat it as a starting point rather than a recommendation.

To use it, copy `.cursor/skills/` into your repo (or into `~/.cursor/skills/` for every project), and Cursor agents will pick the skills up.

Exploratory QA skills for Cursor cloud agents testing a web app in this repo with a shell and a headless browser (Playwright). The tester's job is to break things and report what it finds, judged as a real user would judge it. It never fixes code, opens PRs, or decides product intent. Sessions write their evidence and reports under `qa/` in the working tree, or upload them as run artifacts.

## Skills

| Skill | When it applies |
|---|---|
| `qa-getting-started` | First QA session in the repo, or `qa/SETUP.md` is missing or stale: ask the requester once for target, users, risky categories, quality goals, out-of-scope areas and report destination, and record them. |
| `qa-test-session` | Any request to test the app or a change (PR, branch, preview): charter with the requester, recorded Playwright session behind a risky-probe gate, evidence review, story-style MCOASTER / BUG-NN report, re-tests, and resource cleanup. Includes `references/templates.md` and `references/playwright-harness.md`. |
| `qa-judge-like-a-real-user` | Throughout every session: what counts as a bug (and permission to raise UX, expectation, spec and process gaps), the open first look, the screen-size sweep, comparing against the "before", and the evidence review required before any SAFE verdict. |
| `exploratory-testing-techniques` | Choosing a charter's lenses, finding a fresh angle mid-session, naming the oracle behind a bug, and the report formats. Full heuristic reference in `references/heuristics.md`. |

## Requesting a QA session

Send a brief, not a checklist: the goal (what the user should now see or be able to do, and why); what changed (PR link and exact head SHA, where it shows up, design intent and a "before" screenshot for visual work); who the users are and their situation; the quality bar for this stage; any worries; scope in and out, including findings already waived; and the practicalities (target URL or how to run it, devices, timebox, pre-approved risky probes, how to reach any hard-to-reach state). Leave out item-by-item checks with expected answers and "done when all pass". Expect a five-line charter back before testing starts, and open the key screenshots yourself before acting on a SAFE verdict.

## Credits

- Exploratory testing techniques and heuristic reference adapted with credit from **Dan Ashby**'s *Exploratory Testing Skill*, https://github.com/danashby/Exploratory-Testing-Skill (original README states MIT; the repo has no LICENSE file).
- Heuristic authors as Ashby credits them: FEW HICCUPPS (Michael Bolton & James Bach); SFDIPOT, CRUSSPIC STMPL, MIDTESTD, DUFFSSCRA (James Bach); FCC CUTS VIDS tours and MCOASTER (Michael D. Kelly); testing attacks and RIMGEA (Cem Kaner); Test Heuristics Cheat Sheet data heuristics (Elisabeth Hendrickson, James Lyndsay, Dale Emery); RCRCRC (Karen Johnson); FAILURE (Ben Simo); FIBLOTS and IVECTRAS (Scott Barber); I SLICED UP FUN (Jonathan Kohl); SLIME (Adam Goucher); W5HE (Darren McMillan); SBTM (James & Jon Bach).
- Definition of a bug and the permission to raise UX, expectation, spec and process gaps: Yanqing Cheng, ["What even is a bug anyway?"](https://x.com/YanqingCheng/status/2092296333352652927), drawing on the Bolton/Bach Context-Driven Testing school ("a bug is an inconsistency between the software and what is reasonably expected or desirable").
- QA principles (judge like a real person, briefs not checklists, open look first, evidence review before SAFE, findings only): Yanqing Cheng.
