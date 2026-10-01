# Repository instructions

- Feature PRs target `main`.
- The main checkout is an integration checkout. Do feature work on a dedicated branch in `.claude/worktrees/<slug>`.
- Build a macOS-only client with SwiftUI. Use AppKit where native behavior or measured performance requires it.
- Read `docs/PLAN.md` before implementation. Complete the account and playback feasibility gates before expanding the interface.
- Keep YouTube requests, stream resolution and UI state separate. Network and subprocess work must stay off the main actor.
- Credentials, account responses, browser profiles and signed stream URLs belong outside the repository and diagnostic logs. Use synthetic fixtures in tests.
- Update the documented run and verification commands when the first runnable app lands. Report live-account checks separately from automated tests.
