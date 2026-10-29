# qa-flow-skill

A Claude Code skill for agent-driven QA of iOS app tickets in "Ready for Testing".

- The main session triages the tickets: is the PR merged, is the backend change deployed, does it need a physical device.
- It builds the app once and shares that build across the run.
- One background agent per ticket tests on its own headless simulator with Maestro and reports PASS / FAIL / BLOCKED with screenshots.
- The main session checks the evidence, comments on the ticket, and moves it to Done or Failed, assigned to the PR author.

## Install

```bash
git clone https://github.com/dadiguz/qa-flow-skill.git ~/.claude/skills/qa-flow
cp ~/.claude/skills/qa-flow/project.example.md ~/.claude/skills/qa-flow/project.md
```

Fill in `project.md` with your tracker, repos, build command, test accounts and people. It's gitignored, so private data stays local. Then run `/qa-flow <ticket IDs>`.

## Requirements

- Xcode with an iOS simulator runtime, plus the [Maestro](https://maestro.mobile.dev) CLI
- `gh` CLI authenticated with access to your repos
- An MCP server for your issue tracker (for example Linear)

## Design choices

- Agents never write to the tracker or the code host. Only the main session does, after checking the evidence.
- Every check states what must show and what must not show. A screen that only "loads fine" doesn't pass.
- A failure is re-run once from a clean launch and labeled app bug, test setup or backend before anyone gets a Failed.
- A check that wasn't run blocks a PASS.

MIT License.
