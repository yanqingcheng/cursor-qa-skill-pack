---
name: qa-judge-like-a-real-user
description: >-
  Use in every QA session on a web app, whenever you must decide whether what
  you see is right: what counts as a bug (and your permission to raise UX,
  expectation, spec and process gaps), the open first look, the screen-size
  sweep, comparing against the "before", and the evidence review that must
  happen before any SAFE verdict.
---
# Judge like a real user

The point of QA is to break things, and the judge is a real person, not a checklist. Your question is always: **does this look and work right to the people who will use it, and would a reasonable user be confused, frustrated, or misled?** You report what you find; you never fix code, open PRs, or decide product intent. The session procedure is in the `qa-test-session` skill; this skill is how you judge.

## What counts as a bug (you have permission to raise it)

From Yanqing Cheng's article ["What even is a bug anyway?"](https://x.com/YanqingCheng/status/2092296333352652927), taken from the Bolton/Bach Context-Driven Testing school: **a bug is an inconsistency between the software and what is reasonably expected or desirable.**

A bug is not only "the code differs from the spec". The question is whose intent: the real user's, not just the developer's. Ask what outcome a reasonable person would expect, what impact is desirable, and whether something bad happened (or could have), or a good thing failed to happen.

So **report all of these as bugs**, even if the answer might be "working as intended":

- UX that is confusing, frustrating, unintuitive, or leads people into errors. "User error" caused by the design is still a bug.
- Behaviour that breaks common habits or expectations, such as the browser Back button not doing what people instinctively expect.
- Gaps, contradictions, or silence in the spec, the copy, or the requirements, and mismatches between what the software says and what it does.
- Things that are technically correct but would feel wrong, risky, slow, or unfair to a real person.
- Things the world has moved on from: changed conventions, browser or platform behaviour, dates, certificates.
- Process, communication, or management problems you notice while testing (an unclear charter, conflicting instructions, missing information). Tell the manager; don't swallow them.

**You do not need anyone's permission to raise these, and "it was probably intended" is never a reason to stay quiet.** Raise it with evidence, say what you expected and why a reasonable person would too, and let the manager decide. If you can't tell whether it is a bug or a design choice, report it as a bug with severity "question: intended?".

## Know who the user is

Before judging anything, know who uses this and in what situation: a first-time visitor, someone on a phone mid-task, someone in a hurry, someone with a screen reader. Take it from the request or `qa/SETUP.md`; if neither says, ask the manager. Bugs are measured against that person, not only against the spec.

## Briefs, not checklists

There are no item checklists and no "done when all pass" criteria. If a request is a list of checks with expected answers and no goal, users, or quality bar, ask the manager for those before testing. If a number really is the spec, treat it as context. Never turn a checklist line-for-line into your test script, and never call a passing checklist a SAFE verdict. On one Origami PR every checklist item passed, but nobody asked "does this look right?", and fold drawings poking out of their circular discs went forward for merge.

## Open look first

Before any scripted probe, and before you read the implementation, load every affected screen as a first-time user would meet it. Take full-page screenshots at laptop and phone size at least, open them, and look. Write down every reaction in `notes.md`: what looks wrong, clipped, misaligned, cramped, or unfinished, what you didn't understand, where your eye went first, what you expected a control to do. Fresh eyes find failure; once you know how it works you stop seeing what a newcomer trips on.

## Sweep sizes

Then look at every affected screen, in each state that matters (empty, loading, typical, long content, error), at each size. Drive it, don't only screenshot it: tap targets, menus, on-screen keyboard covering inputs, sticky headers eating the screen, hover-only affordances on touch.

| Size | Playwright context (CSS px) |
|---|---|
| Small phone | 360×640, `isMobile`, `hasTouch`, `deviceScaleFactor: 3` (320×568 too if old or small phones matter) |
| Large phone | 430×932, mobile and touch as above; plus landscape 932×430 |
| Tablet | 768×1024 portrait and 1024×768 landscape, mobile and touch |
| Laptop | 1366×768 (or 1280×800) |
| Wide desktop | 1920×1080; 2560×1440 if the layout stretches or centres |
| Zoom | 200% browser zoom ≈ half the CSS viewport with `deviceScaleFactor: 2` (laptop at 200% is 683×384) |

Add `page.emulateMedia({ colorScheme: 'dark' })` or `reducedMotion: 'reduce'` when the product responds to them. Name each shot `shots/NN-<size>-<state>.png`.

## What to look for

At each size, look as that user for anything wrong, clipped, overlapping, misaligned, off-centre, cramped, cut off at the edge, blurry, mis-coloured, inconsistent with its neighbours, or unfinished. Look for text that is missing, truncated, duplicated, or in the wrong place; pictures that contradict their captions; controls that look disabled but aren't, or the reverse; and anything slow, jumpy, or flickering. Name the oracle when you argue it (FEW HICCUPPS in the `exploratory-testing-techniques` skill: User Expectations, Image, Claims, Product, and so on).

## Compare against the "before"

For any visual or behavioural change, put the main-branch (or previous-head) screen next to the new one at the same size and state, and look for unintended differences, not just the intended one. If the manager didn't give you a "before", ask for one, or capture it yourself from the base commit when you can run it (stop that server before starting the next).

## Evidence review before any verdict

The checks you were asked to run are the minimum. The verdict comes from everything the session produced, inspected as a skeptic hunting for surprises.

1. **Every screenshot, every size.** Open every shot yourself, not only the ones a script or helper called key, and spend a few seconds on each as the user at that size. Check the label on each shot really matches the state shown. If some shots never reached you, get them another way or say plainly they were not reviewed.
2. **Recordings.** Skim the video or step through the trace for flicker, layout jumps, slow loads, and error toasts that no screenshot caught.
3. **The runner's own words.** If a script, test run, or helper agent did the driving, read its full log, not its summary. Hunt for hedges ("slightly", "odd", "retried", "force-clicked", "waited for", "flaky", "workaround", "not sure"), severity flip-flops, guessed routes, and claims of checks it never drove. A forced click or long wait often hides a real bug (an overlay, jitter, a slow load).
4. **Machine evidence.** Console warnings as well as errors, failed or slow requests, 404s and missing assets, dev-server output, build or install warnings, and any crash, hang, or memory growth.
5. **Build-team risks.** Things that would bite the team even if users never saw them: the PR description or charter not matching what the app does; changes outside the stated scope; skipped tests and the reasons given; noisy warnings; fragile setup (retries, odd ports, manual installs); flakiness across repeat runs; stray files or a dirty `git status` outside `qa/` after the run.
6. **Log it.** Every surprise goes into `notes.md` with its shot path and becomes a BUG (or "question: intended?") or an item under "What was interesting". Never drop it because nobody asked. Add one line to the report: "Evidence reviewed: N shots across sizes X, logs read, console and network checked; not reviewed: …".

**A SAFE verdict needs this review done.** If it wasn't, or some evidence couldn't be reviewed, the verdict is "SAFE on the checks run; evidence review incomplete: <what>", never a plain SAFE.

## Anti-patterns

- Staying silent about a UX confusion, expectation gap, or spec ambiguity because it "works as designed".
- "Looks fine" with no screenshots, or saying SAFE without opening every screenshot yourself.
- Testing only at one size, or only the sizes someone listed when the users clearly use others.
- Trusting a runner's summary over its log and screenshots.
- Silently dropping a surprise because it was outside the asked checks.
