# Surfaces and Drivers

The mechanics reference for step 1. Loaded when the QA agent sets up its drive, picks a tool, or captures evidence. The table of surfaces lives in [SKILL.md](SKILL.md); this file deepens it without redefining it.

## Choosing the driver

Closer to the human wins. A real browser over a headless one, a real simulator over device emulation in a desktop browser, a real terminal over a subprocess with piped stdio, and real hardware wherever behavior differs from the simulator (push notifications, camera, biometrics, background execution). When several drivers are available, take the one whose view of the app is closest to a user's eyes and whose actions are closest to a user's hands.

Test the build under test: confirm that the deploy URL, commit, or app version named in the brief is what is actually running — a visible version marker, a fresh build, a just-deployed URL — and record it in the report. QA against the wrong build is worth nothing.

## Web

- **Claude in Chrome** — read the tab context first, open a new tab of your own, and drive with clicks and typing from screenshots. Read the page when a screenshot is ambiguous, read console messages and network requests after every branch, and record a GIF when a finding depends on motion or timing.
- **Codex's in-app browser** — the same discipline: navigate, act, screenshot, read.
- **Playwright CLI** (`playwright-cli`, installed with `@playwright/mcp`) or **Playwright MCP** — `open`, `snapshot`, `click`, `type`, `press`, `screenshot`; prefer `--headed` so you see what the user sees. Use the accessibility snapshot to find controls the way a screen-reader user would, then act on them.

Rules for the drive:

- Two viewports minimum: a desktop width and a phone width (390px is a common phone). A flow that passes at only one width has not passed.
- Wait like a person: watch for the result to appear or the loader to leave, not a fixed sleep. A result that never appears within a patient wait is a finding.
- Fresh profile or cleared storage per root flow, so cached sessions and drafts don't paint over first-use states.
- The console and the network log are part of the surface. An uncaught error, a failed request, or a 4xx or 5xx during a branch is a finding even when the UI looked fine.
- Never bypass: no `evaluate` to set application state, no forged cookie to skip login, no API call in place of the button that makes it. If a bypass is unavoidable — seeding test data, for instance — name it in the report so readers know that stretch was not driven.

## iOS

Boot and manage the Simulator with `xcrun simctl` (`boot`, `install`, `launch`, `io <udid> screenshot`, `openurl` for deep links). Drive the UI with Maestro flows, XCUITest, or a mobile MCP that taps by accessibility label or coordinate; screenshot after every action. Erase the simulator between roots that need a first-run state. Take to real hardware any branch involving push notifications, camera, location, biometrics, or backgrounding — the simulator's answer there is not the user's.

## Android

Boot an emulator, then drive with `adb shell input tap|text|keyevent`, capture with `adb exec-out screencap -p`, or use Maestro for both. Treat the system back button as a first-class action on every screen — Android users press it constantly, and it is the branch web-trained flows forget. Clear app data (`adb shell pm clear <package>`) between roots that need a fresh state.

## Desktop

Use the OS-level computer-use or accessibility automation available to the harness for native apps, and Playwright's Electron support for Electron apps. Screenshot after every action. Cover window resize, the app's own menus and keyboard shortcuts, and what happens on quit with unsaved work.

## CLI

Run in a real terminal session (a pty — `script`, `expect`, or the harness's terminal) so prompts, colors, spinners, and interactive input behave as they do for a user. Drive `--help` and a bare invocation, bad and missing arguments, piped stdin, output when stdout is not a TTY, and Ctrl-C midway through a long operation. Exit codes are part of the surface: a failure that exits 0 is a finding.

## Agent-facing APIs and MCP servers

When the user of the surface is another agent, drive it as that agent would: with only the published docs, schema, or tool descriptions as guidance, through a real client. Branch on what a consuming agent would plausibly try — the documented call, a call with a missing optional field, a wrong type, a stale id, a second call that depends on the first. Errors returned to an agent must be actionable text, not opaque codes; an error an agent cannot recover from is a finding.

## Evidence

Per step: a screenshot or snapshot, the URL or screen identifier, and the console and network state. Per failure: all of that, plus the exact inputs used, plus a short recording when the bug is about motion or timing. Evidence is what lets the implementer reproduce without re-running the drive; a finding without it is a rumor.

## When no driver exists

If no available tool can drive the surface — no simulator on this machine, a device-only feature, credentials the brief didn't provide — the report says exactly which branches were not driven and why. Never downgrade silently to unit tests, code reading, or "should work" and call it QA. Name the block so the person who can lift it does.

## Test data

Use the test accounts and data the brief allows. Never drive against real users' data or real money. Clean up what the drive creates, or mark it clearly so someone can.
