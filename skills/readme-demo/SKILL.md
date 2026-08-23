---
name: readme-demo
description: Design, record, review, optimize, and embed reproducible README demos for developer projects. Use when asked to create or update a project demo, README GIF or MP4, release demo, browser walkthrough, CLI/TUI recording, or visual proof of a feature. Prefer Playwright Screencast for web apps and VHS for terminal surfaces.
---

# README Demo

Create a short, reproducible demonstration that proves the project's core value. Treat it as a presentation-oriented end-to-end test, not a manual screen recording.

## Safety and scope

- Inspect the repository and its local instructions before choosing a scenario.
- Do not expose secrets, personal paths, email addresses, private repository names, tokens, production accounts, or unrelated browser content.
- Use local or disposable demo data. Do not mutate production or send external messages.
- Ask before installing missing system-wide dependencies or modifying the README when the user requested planning or review only.
- Preserve unrelated working-tree changes.

## Workflow

### 1. Understand the product

Inspect the README, package manifests, entry points, existing end-to-end tests, fixtures, and relevant UI code. Determine the value proposition, surface, launch command, URL, test data, authentication, success condition, and cleanup plan.

Do not pretend a native desktop app is supported by Playwright. Explain the missing automation/capture adapter when applicable.

### 2. Propose the storyboard

Before implementing a new demo, propose its audience, one-sentence message, starting state, 3–6 visible actions, observable success condition, target duration, output formats, and cleanup plan.

Prefer 8–15 seconds for a README hero and one feature per demo. Obtain approval when the scenario requires product judgment or material repository changes.

### 3. Choose the capture method

- Read `references/preflight.md` and verify only the dependencies required by the selected capture surfaces. Run launch/render smoke tests; version output alone is insufficient.
- Web app: read `references/playwright-screencast.md`. Prefer Playwright 1.59+ `page.screencast` with precise start/stop. Fall back to context `recordVideo` only when the installed version lacks Screencast.
- CLI/TUI/library example: read `references/terminal-capture.md` and use a committed VHS `.tape` file when practical.
- CLI/agent visibly driving a web app: only when the approved storyboard intentionally combines browser and terminal, read `references/browser-terminal-composite.md`. This is a conditional pattern, not the default web-demo workflow.
- Native desktop: stop and identify an appropriate platform automation/capture tool. Do not silently substitute whole-screen recording.

Reuse existing tests, fixtures, page objects, and stable role/test-id locators. Keep setup outside the recorded interval.

### 4. Author and run the scenario

- Store the deterministic recording script under `scripts/demo/` or `tests/demo/` unless the project has a convention.
- Use assertions or explicit state checks; a video file alone does not prove success.
- Use semantic locators and auto-waiting. Use sleeps only for deliberate presentation pacing.
- Fix the browser viewport, Screencast recording size, and final composite canvas explicitly. Do not accept tool defaults such as 800×500.
- Start capture after the app is ready. Stop after holding the verified final state for 1–2 seconds.
- Restore modified fixtures or data after the run.

### 5. Review the result

Read `references/review-and-export.md`. Inspect extracted frames or a contact sheet before delivery. Confirm the feature is understandable, text is legible, no error or sensitive data appears, pacing is intentional, and the final state proves the assertion.

If review fails, change the scenario and re-record. Do not hide functional failures with editing.

### 6. Export and embed

Keep the original WebM/MP4 until review completes. Generate both MP4 and GIF when useful. Default to 800–960 px width, 10–15 fps GIF, and under 8 MB. Reduce size in this order: fps, width, palette/colors, lossiness, then duration. Verify decoding and size. Use descriptive alt text and follow the repository's asset-location convention.

## Deliverables

Report what the demo proves, scenario path, output paths, duration, dimensions, sizes, validation performed, and remaining privacy or environment limitations.

## Attribution

Adapted from Conor Bronsdon's MIT-licensed `demo-gif-skill`: https://github.com/conorbronsdon/demo-gif-skill. See `references/upstream-notice.md`.
