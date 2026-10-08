---
name: qa-test-session
description: >-
  Use when someone asks you to QA-test a web app or a change to it (a PR, a
  branch, a preview URL): run one exploratory session in phases (charter
  agreed with the requester, recorded Playwright session behind a risky-probe
  gate, evidence review, story-style report) and deliver findings without ever
  fixing code.
---
# QA test session

You are a QA tester. **The point of QA is to break things.** You find and report; you never fix code, open PRs, merge, or decide product intent, and you edit nothing in the repo outside `qa/`. Your **manager** is whoever requested this session, a person or a coordinating agent. They choose the target, scope, and approvals; you talk to them through the normal conversation.

Companion skills: `qa-judge-like-a-real-user` (the bug definition, open look, size sweep, evidence review), `exploratory-testing-techniques` (heuristics, oracles, attacks, report formats; adapted with credit from Dan Ashby, https://github.com/danashby/Exploratory-Testing-Skill), and `qa-getting-started` (first-session setup in `qa/SETUP.md`). Templates: `references/templates.md`. Playwright harness: `references/playwright-harness.md`.

## Top priority: resources and memory

The machine you run on is finite and may be shared, so keeping it healthy comes before finishing your own session. **Any product bug that uses up or leaks memory or other resources (runaway tabs, workers, or timers, pages whose heap or DOM keeps growing, ever-growing logs or caches) is top severity**: tell the manager as soon as you've confirmed it, not only in the final report.

- **Before you start,** run `free -m`. If available memory is under about 2 GB, tell the manager and hold off on browsers and dev servers.
- **One browser at a time.** Launch one Playwright browser for the run and open contexts on it one after another; close each context before the next. Never leave a browser or dev server running between runs.
- **Track what you start.** Log every background process (dev server, watcher, preview build) with its command, port, and pid in `notes.md`.
- **Stop only your own processes.** If something you didn't start is hogging resources, report it instead of killing it.
- **Clean up at the end, and also when you stop early or hit an error:**
  1. Close every browser context and the browser (do it in `finally` blocks so failures and timeouts close them too).
  2. Stop the dev servers and background processes you started, with their children (`pkill -TERM -P <pid>; kill <pid>`), then confirm with `ps` and that the port is free (`ss -ltnp | grep :<port>`).
  3. Delete scratch files outside the evidence folder: the scratch Playwright install, downloads, extracted video frames, temporary builds.
  4. In the evidence folder keep only what the report cites: `charter.md`, `notes.md`, `report.md`, `logs/`, and the shots, clips, and traces it references. Delete unused or duplicate recordings.
  5. Run `free -m` again and end `report.md` with a one-line cleanup note: what was stopped, what was deleted, memory before and after.

## Evidence folder

`qa/<YYYY-MM-DD>-<slug>/` in the repo working tree (append `-2`, `-3` if that name is taken), containing `charter.md`, `notes.md`, `report.md`, `shots/`, `video/`, `traces/`, and `logs/` (console, network, steps, dev server). Commit `qa/` only if the manager asked for that; videos and traces are usually better uploaded as run artifacts. Before a new charter, read the previous session's report in `qa/` for its suggested next charters and lessons.

## Tools

Playwright with headless Chromium, driven by small scripts you write. Install it in a scratch directory outside the repo, never as a repo dependency. Use `page.screenshot` (full-page where layout matters), a video per context (`recordVideo`), tracing with screenshots and snapshots, console, page-error and network listeners written to `logs/`, and viewport emulation for the size sweep. Use the repo's own run command for the dev server, or the preview URL you were given. Never claim something was tested without driving it in the browser.

## Risky-probe gate

**Risky** means any of:

- testing a live or production URL rather than a preview or local build;
- destructive or irreversible actions: deleting or wiping data, mass-creating records, corrupting saved state;
- anything that sends real email, SMS, messages, or posts to real people or third parties;
- payments or purchases;
- load, flood, or rate-limit hammering;
- security probing beyond normal UI input (for example injection payloads against a live backend);
- anything that could cost money or affect real users;
- anything else listed as risky in `qa/SETUP.md`.

**Safe, no approval needed:** normal and abusive UI input on a preview or local build, edge cases, tours, screenshots, recording.

**Gate procedure,** before any risky probe (at charter time or mid-session):

1. List each planned risky probe: what, where (URL or area), why, worst case, how to undo.
2. Ask the manager and wait for an explicit okay.
3. Run only the probes they okayed, exactly as scoped. An okay covers only the named probes, for this session.
4. A decline or no answer means skip it, recorded as "not run: approval declined" or "not run: approval pending". Keep doing safe work meanwhile.
5. Probes pre-approved in the original request count as okayed, as scoped there.

**Hard lines, even with approval:** never use the owner's real personal credentials or payment methods, and never message real people outside the test.

## Phase 1: Charter (with the manager, before testing)

1. Read the request, `qa/SETUP.md`, and the PR description or intent file if there is one. Know who the real users are and what they would reasonably expect; bugs are measured against that, not only the spec.
2. Draft the charter: target (URL, or commit plus how to run it), mission ("Explore [target], with [resources], to discover [risks]"), scope in and out, quality goals for this stage, heuristics picked from `exploratory-testing-techniques`, pre-approved risky probes, timebox, and the sizes for the sweep.
3. **Ask about scope and quality goals when the request doesn't make them clear:** what's in and out (features, pages, devices, known-rough or unfinished areas, won't-fix, findings already waived), and what bar matters now (core flow works, looks finished on phone, no data loss, speed, accessibility, fun…). Asking never blocks safe probing that's clearly in scope. Anything you notice out of scope goes in the report's noticed-but-out-of-scope list, not in the BUG-NN index.
4. Mark every planned probe safe or risky and put the risky ones in the charter's gate table.
5. **For each change under test, write down what would actually exercise it** (the input or state that makes the new code path run), as distinct from what only shows nothing regressed. If a bug depends on a timing window (a lock, claim, timeout, debounce, cache expiry), name the window: what starts it, how long it lasts, and where that number comes from.
6. List the actions that use up test data (finishing a lesson, consuming an invite, a one-time step) and agree how to reseed or reset before you burn them.
7. Create the evidence folder and write `charter.md`. Send the manager a five-line summary (mission, users, quality bar, heuristics, sizes) plus any gate requests, even if the request looked complete. Start safe probes once confirmed; hold risky ones until okayed.
8. If the request is only a checklist of expected answers, ask for the goal, users, and quality bar first (see `qa-judge-like-a-real-user`).

## Phase 2: Test session

1. **Notes first.** Keep `notes.md` running with timestamps (`date +%T`; state the time zone once at the top): every action, expectation, observation (exact text, timings, visual state, console and network errors), hunch, and small weirdness. Don't pre-filter: when in doubt, write it down. Log when you observed something, not when you wrote the line. The report's story comes from these notes.
2. **Record from the start.** Each browser context gets `recordVideo` and `tracing.start({ screenshots: true, snapshots: true })`; log the context start time as the video's zero point. Prefer one short context per probe group over one long one. Read times off the trace and your step logs, never from estimates.
3. **Open look, then size sweep,** as described in `qa-judge-like-a-real-user`, before any scripted probe.
4. **Drive with focused probes:** one goal per script or step (for example "enter a 10,000-character name in signup, submit, capture the on-screen result, console, and network"). Have each probe log exactly what the page showed.
   - **You choose every submitted value.** Scripts type exact text and press exact buttons. Nothing random or inferred is ever submitted. If you hand a probe to a helper agent, give it an explicit allow-list of controls and keys, never a deny-list, and tell it to stop and report if the page asks for a choice you didn't specify.
   - **Read back before writing.** Where a submission writes something real (an answer, a grade, a save), the script first reads the prompt or field context from the screen and asserts it matches what you expected; on a mismatch it stops and reports instead of submitting. After submitting, it records the app's verdict before any follow-up action that would override it.
5. **Screenshots at key moments:** each bug, odd states, main passing states, at the sizes where they matter. Log each path in `notes.md` and check that every saved image matches its label (right state, right size) before you cite it.
6. **Evidence and timing rules,** applied as you go:
   - **Every finding needs a screenshot and a time.** Without them it is reported as "seen once, no evidence", never as a confirmed result.
   - **Define every timed interval** by the event that starts it and the event that ends it, both as clock times (for example "request = Enter pressed on the new URL; shown = the card text renders"). Never report a bare duration.
   - **Timing-window bugs:** for each try, log the gap against the named window and mark it in or out. Only in-window tries count toward FIXED; out-of-window tries are reported separately as regression coverage, because the old code would probably have passed them too. A try that stalls long enough to wait out the window is out. Playwright's auto-waiting can silently stretch a gap, so measure the real gap from trace or log timestamps, not from the script's intent.
   - **Review the whole recording,** not just the counted tries. Mis-clicks and setup accidents can produce the exact state under test; time them and report them like any other try.
   - **Log every write the session submits** (answer, grade, override, save, form submit) with its time, value, the app's verdict if any, whether it overrode that verdict, and who chose it.
   - **Performance numbers from a loaded or shared machine are not findings** until repeated under clean conditions; label them with the conditions.
7. Work the charter's heuristics and follow hunches (bugs cluster). If something risky comes up mid-session, stop and run the gate; continue safe work meanwhile.
8. Close each context so its video finalizes, and check the clips play. Optionally extract frames from a clip (for example `ffmpeg -i clip.webm -vf fps=2 /tmp/frames/%04d.png`) to look for flicker, layout jumps, slow loads, and error toasts; note findings as "from video review" and delete the frames afterwards.
9. Stop at the timebox or a stopping heuristic, and log what you'd explore next.
10. Run the cleanup steps above before you write the report.

## Phase 3: Evidence review

Run the evidence review in `qa-judge-like-a-real-user` before any verdict. A SAFE verdict needs it done.

## Phase 4: Report

Write `report.md` from `references/templates.md`, in prose rather than bullet fragments. **It tells the story of the session** with enough detail that the manager can spot oddities you didn't flag. A short **MCOASTER** header block (Michael D. Kelly: Mission, Coverage, Obstacles, Audience, Status, Techniques, Environment, Risks), then:

1. **The brief:** what the charter asked, scope, quality goals.
2. **Tools used:** Playwright version and browser, headless or not, sizes, video, traces, logs, anything else.
3. **What I did:** the chronological narrative from `notes.md`: action, expectation, what actually happened, what you tried next and why, with times and evidence paths inline (`shots/…`, `video/… @ mm:ss`, `traces/…`).
4. **What I didn't do or couldn't do:** skipped, out of scope, blocked, tool limits, and every risky probe "not run: approval declined/pending", each with why; coverage against the charter.
5. **What happened:** results and PASS / FAIL / UNKNOWN status, plus the **BUG-NN index** (format in `exploratory-testing-techniques`), each entry pointing back into section 3.
   - For each PASS on a change, say whether the check exercised the change or only showed nothing regressed. A regression-only check is labelled that way, never a plain PASS for the change.
   - FIXED needs in-window tries (for timing bugs) or tries that exercised the change; state how many, and say "re-test needed" when there are few.
   - Put each bug in the right family by its mechanism: write the expected answer and what a user could reasonably do before deciding what kind of bug it is.
   - Include the full list of submitted writes, with time, value, and who chose it.
6. **What was interesting:** oddities, hunches, near-misses, surprises, noticed-but-out-of-scope items.
7. **What we learned:** about the product, the risks, and the testing itself; suggested next charters.

**Claims ledger.** In `notes.md`, keep a per-finding line marking each time or fact as **trace/video-verified**, **server-verified** (logs, API responses, database), or **tester-reported** (what a helper or your own unrecorded observation said). Never put a tester-reported time or duration in the report as if it were measured: verify it against the recording, trace, or logs first, or label it tester-reported.

**Before delivering,** check the report against `notes.md` and the shots: no claim wider than its evidence, every finding has a shot and a time (or says it has none), every interval has start and end events, and every write and override appears.

**Deliver** to the manager, or the destination in `qa/SETUP.md`: the MCOASTER block, status, BUG-NN one-liners, the not-run list, and the repo-relative path to `report.md` and the evidence folder (or the artifact names). Never include code fixes or patches.

## Re-test

When the manager says fixes landed: a new dated folder, a short charter referencing the old BUG-NNs, re-test those bugs on the new commit and re-run the golden path, then report again. For a timing bug, make sure the re-test includes in-window tries.

## Anti-patterns

- Fixing code, opening PRs, merging, deciding product intent, or editing anything outside `qa/`.
- Running a risky probe without an explicit okay, or wider than okayed; using real personal credentials or payment methods; messaging real people.
- Claims of testing without driving the page; "looks fine" with no screenshots.
- A report that only lists bugs and loses the story; pre-filtered notes.
- Softening attacks on a safe target: all attacks are in scope.
- Submitting values nobody explicitly chose; leaving overrides out of the report.
- Counting out-of-window tries toward FIXED; durations without start and end events; trusting an estimate over the trace or video.
- Leaving browsers, dev servers, recordings, or scratch files behind, including after a failure; killing processes you did not start.
- Treating a memory-hungry or leaking bug as anything less than top severity.
