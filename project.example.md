# qa-flow project config

Copy to `project.md` (kept out of version control) and fill in. qa-flow reads it on every run.

## Tracker
- Tool / MCP: <e.g. Linear MCP `mcp__linear__*`>
- Team / project: <key>
- Statuses: testing = <Ready for Testing>, done = <Done>, failed = <Failed>

## Code
- App repo: <owner/app> · default branch: <main>
- Backend repo: <owner/api> (optional)
- Deploy check (backend merge live in the test environment): <command, e.g. `gh run list -R owner/api --branch main` and a green deploy job after mergedAt>

## Build
- Worktree path: <path>
- Command: <xcodebuild -scheme "<scheme>" -configuration "<config>" -sdk iphonesimulator CODE_SIGN_IDENTITY="-" -derivedDataPath <scratch>/qa/build/DerivedData>
- Bundle id: <com.example.app>
- Simulator: <iPhone model>, latest iOS runtime · concurrency cap: <1>
- Extra build gotchas: <optional>

## People
- PR author map (code host login → tracker user): <login → name>
- Contact for contradictions with earlier tickets or design: <@handle>

## Accounts
- Test accounts (login, OTP or how to sign in): <list>
- Forbidden accounts (never add, invite, follow or message): <list>

## Comments
- Language: <English>
- Style: <very short, plain language, no code names; or path to a style skill>

## After a FAIL by the user
- Fix skill to launch: <optional, e.g. /ticket-flow>

## Notes
- <anything that can clash with this flow, e.g. another automation that reassigns tickets in the testing status>
