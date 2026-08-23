# Playwright web capture

## Selection

1. Check the installed Playwright version and configuration.
2. Prefer the project's package and conventions.
3. Use Playwright 1.59+ Screencast when available. Otherwise use `recordVideo` and close the context before accessing the video.
4. Select one explicit canvas, for example 1280×720, and use it for viewport, recording, and final composition.

## Screencast pattern

Adapt exact API syntax to the installed language binding and confirm it against local types or official documentation.

```ts
const captureSize = { width: 1280, height: 720 };
await page.setViewportSize(captureSize);
await page.goto(baseURL);
await prepareDemoState(page);

await page.screencast.start({
  path: 'docs/demo.webm',
  size: captureSize,
});
const actions = await page.screencast.showActions({ position: 'top-right' });
await page.screencast.showChapter('Core workflow', {
  description: 'A short description of the visible goal',
  duration: 800,
});

await page.getByRole('button', { name: 'Create' }).click();
await expect(page.getByText('Created')).toBeVisible();
await page.waitForTimeout(1500); // intentional final-state hold

await actions.dispose();
await page.screencast.stop();
```

## Rules

- Set viewport and Screencast size explicitly and match the final composite canvas. Confirm exact option names against the installed Playwright types.
- Inspect the raw recording dimensions with `ffprobe`; do not assume requested dimensions were honored.
- Disable unrelated notifications, extensions, password managers, and browser chrome.
- Seed deterministic fake data before recording.
- Prefer role, label, and test-id locators.
- Wait for meaningful state, not guessed duration.
- Keep login, server startup, fixture creation, and debugging outside capture.
- Decide explicitly whether test animation-reduction settings are appropriate for the demo.

For older Playwright versions, use a context with matching `recordVideo.size` and viewport. Never omit either size. The entire context lifetime is recorded, so separate setup from the recorded context when possible.

Documentation:

- https://playwright.dev/docs/release-notes
- https://playwright.dev/docs/videos
