---
name: qa-flow
description: Agent-driven QA for tickets in "Ready for Testing" on an iOS app — triage, one shared build, one background agent per ticket testing on its own headless simulator with Maestro, PASS/FAIL/BLOCKED with evidence; the main session verifies the evidence, comments on the ticket and moves it to Done or Failed (assigned to the PR author). Use when the user says "QA these tickets", "test my Ready for Testing", "lanza QA", passes ticket IDs to test, or invokes /qa-flow.
---

# QA flow

The main session orchestrates and owns every tracker write; background agents only test and report. Talk to the user in their language, short bullets.

**Project config:** everything specific to the project (tracker, repos, build, accounts, people, statuses) lives in `project.md` next to this file. Read it first. If it's missing, copy `project.example.md`, fill it with the user, and stop until it's done. Never guess a value that belongs there.

**Input:** ticket IDs, or "mine" = tickets in the testing status assigned to the user.

## 0. Triage (main session, before any agent)

For each ticket:
1. Read it in full: description, comments (implementer's "What to test" and "Decisions"), attachments (PR links).
2. **Author = PR author on the code host, not the ticket assignee** (the assignee is usually the tester). `gh pr view <url> --json author,state,mergedAt`, then map the login with the author table in `project.md`; unknown login → look it up in the tracker and confirm.
3. **Is it testable now?**
   - Every PR merged to the default branch. Not merged → skip, tell the user.
   - Backend PR → its merge is deployed to the test environment (deploy check in `project.md`). Not yet → wait or skip.
   - Needs a physical device (real camera, push delivery, real GPS, real payments, store purchase) → **BLOCKED** up front, no agent.
4. Classify how it's tested: **UI** (simulator + Maestro), **assistant/chat** (ask in the app and judge the answer), **backend-only** (call the test API with a test user, or observe it through the UI).

Show the user the triage as one line per ticket (ID, author, how it's tested, or why skipped) and go.

## 1. One shared build

Don't make every agent compile. Once per run:
- A detached worktree of the app repo at the default branch (`git worktree add --detach <path> origin/<default>`). Never touch the main checkout, never create or switch branches.
- Build with the command in `project.md` for the iOS simulator, ad-hoc signed (`CODE_SIGN_IDENTITY="-"`; unsigned builds with embedded frameworks may not launch), pointing at the test environment.
- Write `BUILD.md` in the scratch folder: commit SHA, absolute `.app` path, build command. Every ticket's merge must be in that commit.

## 2. One background agent per ticket

Concurrency: the cap in `project.md` (default 1 simulator, sequential). Give each agent all of this (it knows only what you write):

- **Ticket:** ID, full expected state / constraints / user journey, the original bug report if it's a bug, the implementer's "What to test", related tickets, how to test it (from triage), the PR diff link.
- **App:** path of the shared `.app` + commit. Don't rebuild. If the build looks wrong, stop and report.
- **Simulator:** its own, by UDID, never `booted` (another session may have one booted). Headless only (`xcrun simctl`, `maestro --device <udid>`); no tool that opens a simulator window in the user's app. Shut it down at the end; delete it only if the run is over. System permission prompts: tap the button by text, not coordinates.
- **Accounts:** only the test accounts in `project.md`. Never the forbidden accounts listed there, never invite, follow or message anyone else. No real money.
- **Scratch:** everything (UDID, flows, screenshots, videos) in `<scratch>/qa/<ID>/`.
- **What to do:**
  1. If it's a bug: check #1 is the reporter's steps as written (same entry point, same kind of account). The implementer's "What to test" adds checks; it never replaces these.
  2. Turn each expected-state bullet and each constraint into a numbered check. For each, write what the screen **must** show (exact text, value or count) and what must **not** show (the old behavior, an error, still on the previous screen). Judge against `maestro hierarchy` text or a critical read of the screenshot. "Loads / no crash / looks right" is never a pass.
  3. Prepare the data a check needs through the app with test accounts. Never by writing to the database.
  4. Run every check. One screenshot per check; Maestro `startRecording` for motion and sheets (`simctl recordVideo` may miss sheets).
  5. One edge case, the most relevant of: kill and relaunch (does it persist?), background then foreground (does live data come back?), back mid-flow, double-tap the main button, a second test account sees the change.
- **Don't:** write to the tracker, the code host or code; change anything outside the test data.
- **Report** (English, short):
  - Verdict: `PASS` / `FAIL` / `BLOCKED`.
  - Per check: pass / fail / NOT RUN (with why) + evidence path. Any NOT RUN means the verdict can't be PASS.
  - FAIL: before calling it, relaunch the app and re-run the failing check once. Report "reproduced 2/2" or "1/2 = intermittent", label it **app bug / my setup / test backend**, then steps, expected vs actual, evidence.
  - BLOCKED: what stopped it and what's needed.
  - Draft ticket comment in the comment style from `project.md`.

## 3. When an agent reports (main session)

1. Open the evidence. A PASS without a screenshot per check isn't a PASS; a FAIL labeled "my setup" or "test backend" isn't a FAIL. If unsure, send the user the screenshot and ask.
2. One **new** comment on the ticket (never edit old ones), comment style from `project.md`, evidence embedded:
   - **PASS** → "Passed QA" + what was checked → done status.
   - **FAIL** (app bug) → "Failed QA" + what fails, steps, expected / actual → failed status, assign the **PR author**.
     - Author is the user → assign the user and, if `project.md` names a fix skill, launch it with the failing steps in the brief.
   - **BLOCKED** → no tracker change; tell the user what's needed.
   - If a result contradicts an earlier ticket or the design, say so and tag the contact in `project.md`.
3. Send the user 1–2 key screenshots per FAIL.

## 4. End

- Delete the QA worktree and every simulator this run created.
- One table for the user: ID · verdict · one line · link. Then one concrete next action.

## Never

- Use real or forbidden accounts, invite anyone outside the test list, move real money.
- Mark Done without evidence, or Failed because of a setup problem.
- Let agents write to the tracker or the code host.
- Open the simulator window in the user's app or use `booted`.
