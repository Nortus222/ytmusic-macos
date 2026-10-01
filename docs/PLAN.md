# Native YouTube Music for macOS

Discussion and progress: [wayfinder issue #1](https://github.com/Nortus222/ytmusic-macos/issues/1). These phases are proposals; authentication, audio playback and distribution choices remain subject to feasibility results.

## Product decision

Build a macOS-only SwiftUI app, with AppKit for native window behavior or measured UI bottlenecks. Use YouTube Music Desktop as a behavior reference and ytfast as a reference for service integration. Keep playback independent of the interface so the feasibility experiment can determine its implementation.

The first milestone answers whether a real account can browse its playlists and play, seek and advance through tracks reliably. It is not a promise of complete YouTube Music parity.

## Initial scope

- One account, with an explicit connection and reconnect flow.
- Audio playback, search, library playlists and playlist tracks.
- Queue, previous/next, play/pause, seeking and volume.
- Native sidebar, search, track list and persistent bottom player.
- Media keys, Now Playing metadata and playback after closing the window. Quit ends playback.
- Cached metadata and artwork, refreshed after the interface appears.

Defer playlist editing, likes, lyrics, videos, downloads, podcasts, plugins and cross-platform support until this milestone works. Cached metadata is not offline audio playback.

## Proposed implementation

Use a small set of components rather than a cross-platform framework:

- The service client owns YouTube Music requests, response parsing and session validation.
- The resolver turns a track ID into a playable stream and tracks its expiry. Start by testing the installed yt-dlp and Deno tools.
- The player owns audio state, seeking and transitions. Test AVPlayer/AVQueuePlayer with authenticated AAC streams before selecting them. Test mpv if native playback fails the compatibility or transition checks.
- The app model owns navigation and the queue. Late results must not replace a newer page, track or account.
- The cache stores account-scoped metadata and bounded artwork. Credentials stay in Keychain or an explicitly selected external session source, never in the cache.

No app-operated server is needed for the first milestone. Authentication and stream extraction remain unofficial integration dependencies that may change.

## Phase 1: prove service access

1. Establish a Swift package and a small command-line probe sharing the service code with the future app.
2. Fetch and parse public search and a playlist to validate metadata access independently of authentication.
3. Evaluate a macOS account connection method. Investigate explicit browser-session import with user-selected browser/profile and macOS Keychain access. Do not silently inspect every browser profile.
4. Confirm the account using an authenticated request, then fetch the user's library and one complete playlist, including pagination.
5. Check playback authentication independently. A working library request does not prove stream extraction works.

Google's supported system-browser OAuth flow does not automatically supply the browser session used by private YouTube Music endpoints. Embedded Google sign-in is an experiment, not an assumed solution.

Completion evidence: a redacted probe report showing public access, confirmed account access, playlist pagination and the chosen session/reconnect mechanism. Real credentials and API payloads stay local. If account connection cannot be made practical, record the blocker before building the full interface.

## Phase 2: prove audio playback

1. Resolve a real track with yt-dlp using argument arrays, bounded execution and cancellation. Keep authenticated cookie files private and remove temporary files after use.
2. Try AAC audio in AVPlayer. Check required headers, URL expiry, seeking and behavior after sleep/wake.
3. Play a queue through at least three track transitions. Prepare the next stream before the current track ends, respecting expiry and cancellation.
4. Prevent a late resolver result from playing a superseded track. Track changes invalidate previous preparation work.
5. Exercise a failed stream, expired session and network interruption. Show actionable errors and retain queue state; retries must be bounded.
6. Measure cold resolution separately from prepared playback and track transitions. Choose the player from results, not assumed language performance.

Completion evidence: local results for real audio, seeking, three transitions, cancellation and recovery, plus a decision about AVPlayer versus mpv. Record the actual codec and account quality offered. Do not claim gapless playback without measuring it.

## Phase 3: build the native app

1. Add sidebar navigation, search, library playlists, paginated tracks and the bottom player.
2. Connect the queue to playlist playback and search results. Keep selection and current playback visible.
3. Add keyboard navigation, Command-K search, media commands and Now Playing metadata.
4. Keep playback alive when the last window closes; reopening restores the window and current queue. Quit stops playback and child processes.
5. Add explicit account status and reconnect controls. Changing accounts clears account-scoped UI and cache state.
6. Restore cached content at launch and refresh without replacing newer navigation results. Bound artwork caching and load visible rows progressively.

Completion evidence: a runnable app completing connect → library → playlist → play → seek → next, plus search, window reopening and keyboard/media control checks. Include VoiceOver and light/dark appearance checks.

## Phase 4: measure and package

Measure release builds on the same Mac with the same account, playlist and network. Record hardware, macOS, tool versions and sample counts. Include subprocesses when reporting memory and CPU.

Provisional goals, subject to the feasibility results:

| Measurement | Goal or required evidence |
| --- | --- |
| First usable window with cached content | Under 1 second on the development Mac |
| Local UI response | No network wait for navigation, queue edits or play/pause |
| Prepared song start | Under 1 second after selection |
| Cold song start | Report median and p95; investigate delays above 3 seconds |
| Track transition | Measure audible gap; target no perceptible gap |
| Idle resource use | No continuous redraw or polling loop; report CPU and total memory |
| Background playback | Report CPU/memory with the window closed |

Collect a baseline from YouTube Music Desktop with equivalent workloads if it is available. Treat language/framework comparisons as unverified until measured.

Package a macOS app with clear resolver dependencies. Signing and notarization depend on the owner's available Apple credentials. Avoid runtime Homebrew installation; decide whether to bundle dependencies only after checking distribution licenses and update requirements.

Completion evidence: one full release build, recorded performance results, a documented launch command or app bundle, and an explicit list of checks that still require a real account or signing credentials.

## Verification

Use targeted tests for response parsing and pagination, session rejection, expired stream reuse, stale asynchronous results, queue advancement and cancellation. Use synthetic service responses and a fake resolver for automated tests. Do not replace the live playback checks with mocks.

Add executable build/test commands when the Swift project exists. Run targeted tests during implementation and one full release build at the end of the implementation batch. A successful build does not establish passing tests or working authenticated playback.

## Source references

- [ytfast](https://github.com/MayberryDT/ytfast): Rust/egui Linux client using private YouTube Music endpoints.
- [ytfast integration notes](https://github.com/MayberryDT/ytfast/blob/main/docs/integration.md): session handling, stream expiry, pre-resolution and reported cold-resolution delays of 4–10 seconds.
- [YouTube Music Desktop](https://github.com/ytmdesktop/ytmdesktop): Electron desktop behavior reference.
- [Pear Desktop](https://github.com/pear-devs/pear-desktop): the project formerly named th-ch/youtube-music.
- [Google installed-app authorization](https://developers.google.com/identity/protocols/oauth2/native-app): system-browser OAuth and embedded-user-agent limitations.
- [yt-dlp YouTube extractor notes](https://github.com/yt-dlp/yt-dlp/wiki/Extractors#youtube): authentication and stream-extraction requirements.

The cited projects are references, not macOS performance measurements. Preserve source attribution and licenses if implementation code is reused.
