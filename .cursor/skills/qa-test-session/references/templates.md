# Session templates

Used by the `qa-test-session` skill. Copy into the evidence folder (`qa/<YYYY-MM-DD>-<slug>/`) and fill in. Report formats follow the `exploratory-testing-techniques` skill (adapted with credit from Dan Ashby, https://github.com/danashby/Exploratory-Testing-Skill; MCOASTER by Michael D. Kelly, RIMGEA by Cem Kaner, as credited there).

## charter.md

```markdown
# Charter: <project> / <YYYY-MM-DD>-<slug>
- Manager: <who requested this>
- Target: <URL (preview | local | live)> or <commit SHA + run command>
- Change under test: <PR / branch / head SHA; one plain paragraph on what changed and where it shows up>
- Before: <main-branch screenshot paths, or how to capture them>
- Users and their expectations: <who, in what situation, what they'd reasonably expect>
- Mission: Explore <target>, with <resources/data/tools>, to discover <information about risks>.
- Scope in: <flows, pages, features>
- Scope out: <explicitly not tested; findings already waived>
- Quality goals: <the bar for this stage, e.g. "must look finished on phone and desktop">
- What exercises each change: <input or state that makes the new code path run>
- Timing windows: <what starts it, how long, where the number comes from> (or "none")
- Test data used up: <actions that consume data; reseed/reset plan>
- Heuristics: <oracles, e.g. FEW HICCUPPS Claims + User Expectations>; <tours>; <attacks>; <data heuristics>
- Sizes: <small phone 360x640, large phone 430x932, tablet 768x1024 + landscape, laptop 1366x768, wide 1920x1080, 200% zoom>
- Timebox: <e.g. 90 min>
- Pre-approved risky probes: <from the request, or "none">
- Risky probes requested (gate):
  | # | What | Where | Why | Worst case | Undo | Status (okayed / declined / pending) |
  |---|------|-------|-----|------------|------|------|
- Confirmed by manager: <yes / time>
```

## notes.md (running, timestamped)

```markdown
Times are UTC (date +%T).
09:01:30 free -m: 5400 MB available. dev server `npm run dev` started, port 5173, pid 4121
09:02:10 [ctx laptop] video t0; trace started
09:02:40 [action] opened <url> at 1366x768; expected <x>; saw <exact text / state>; load felt ~3s (tester-reported)
09:04:05 [odd] button label flickers on hover; unsure if bug. shots/03-laptop-flicker.png
09:06:30 [bug?] 10k-char name accepted, layout breaks. shots/04-small-phone-longname.png, video/<file>.webm @ 04:20
09:06:31 [ledger] BUG-01 time: trace-verified (traces/small-phone.zip)
09:07:00 [write] 09:06:58 submitted name="<10k chars>" via Save; app said "Saved"; chosen by: me
09:07:10 [next] trying the same input in profile edit, since bugs cluster
```

## report.md

Prose, not bullet fragments (the header block and bug index excepted).

```markdown
# QA report: <project> / <YYYY-MM-DD>-<slug>
Evidence: qa/<YYYY-MM-DD>-<slug>/ (artifacts: <names, if uploaded>)

> **MCOASTER**
> Mission: <charter sentence> · Coverage: <areas explored> · Obstacles: <blockers> ·
> Audience: <manager> · Status: PASS / FAIL / UNKNOWN (one line why) ·
> Techniques: <heuristics used> · Environment: <URL or commit, browser and Playwright version, sizes> ·
> Risks: <found / confirmed / remaining>

## 1. The brief
What the charter asked for, what was in and out of scope, and the quality goals.

## 2. Tools used
Playwright (version, browser, headless), screenshots, video per context, traces, console and network logs, sizes emulated, anything else.

## 3. What I did
The story in time order, from notes.md. For each step: what I did, what I expected, what actually happened (exact text, timings, visual state, console and network errors), what I tried next and why, and any hesitation or hunch. Inline evidence, e.g. "At 09:06 (video/<file>.webm @ 04:20, shots/04-small-phone-longname.png) …". Mark BUG-NN where each bug appears.

## 4. What I didn't do or couldn't do
Skipped, out of scope, blocked, tool limits, and why. Risky probes: "<probe>: not run: approval declined" / "not run: approval pending". Coverage against the charter.

## 5. What happened
Results and overall status. Evidence reviewed: N shots across sizes X, logs read, console and network checked; not reviewed: …

BUG-01: <title>
- Criterion: <oracle / standard>
- Severity: CRITICAL / HIGH / MEDIUM / LOW / question: intended?
- Screen / area: <…> at <sizes>
- Repro steps: 1. … 2. … 3. …
- Expected vs actual: <…> / <…>
- Evidence: shots/NN-<size>-<slug>.png; video/<file>.webm @ mm:ss; traces/<name>.zip; see §3 at HH:MM
- Expected behaviour / likely area (advisory, the tester does not fix): <…>

| ID | Criterion | Severity | Screen | Status |
|----|-----------|----------|--------|--------|

Submitted writes: | Time | Action | Value | App verdict | Overridden? | Chosen by |

Recommendations: Priority 1 / 2 / 3 groupings for the owner to triage (advisory; no code fixes).

## 6. What was interesting
Oddities, hunches, near-misses, surprises, and noticed-but-out-of-scope items, even if not judged bugs.

## 7. What we learned
About the product, the risks, and the testing itself. Suggested next charters.

Cleanup: stopped <processes>; deleted <scratch>; memory available before <N> MB, after <M> MB.
```
