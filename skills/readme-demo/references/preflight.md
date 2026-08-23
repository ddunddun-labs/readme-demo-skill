# Capture preflight

Run preflight after selecting the capture surfaces and before writing the final demo. Adapt commands to the host shell and project package manager. Do not require terminal dependencies for an ordinary browser-only demo.

## Dependency matrix

| Selected surface | Required checks |
| --- | --- |
| Browser only | Playwright package, Playwright Chromium launch, ffmpeg, ffprobe |
| Terminal only | vhs, ttyd, VHS Chromium render smoke test, ffmpeg, ffprobe |
| Browser + terminal composite | All browser and terminal checks |

## Binary checks

```sh
command -v vhs
command -v ttyd
command -v ffmpeg
command -v ffprobe
vhs --version
ttyd --version
ffmpeg -version
ffprobe -version
npx playwright --version
```

Run only the rows required by the selected surface. For a combined browser/terminal demo, all checks are required.

## Playwright Chromium launch check

Version output does not prove the browser binary exists. Launch it:

```sh
node - <<'NODE'
const { chromium } = require('@playwright/test');
(async () => {
  const browser = await chromium.launch({ headless: true });
  console.log(`Playwright Chromium: ${await browser.version()}`);
  await browser.close();
})().catch((error) => {
  console.error(error);
  process.exit(1);
});
NODE
```

If the project imports `playwright` rather than `@playwright/test`, use that package. Install Chromium only after confirming installation is in scope.

## VHS render smoke test

Checking `vhs --version` is insufficient because rendering also launches ttyd, Chromium, and ffmpeg. Render a disposable tape:

```sh
preflight_dir="$(mktemp -d)"
printf '%s\n' \
  'Output "'$preflight_dir'/smoke.gif"' \
  'Set Width 640' \
  'Set Height 360' \
  'Set Framerate 8' \
  'Type "echo preflight-ok"' \
  'Enter' \
  'Sleep 500ms' > "$preflight_dir/smoke.tape"
vhs "$preflight_dir/smoke.tape"
ffprobe -v error -show_entries stream=width,height -of default=nw=1 "$preflight_dir/smoke.gif"
```

Delete the disposable directory after inspection. A successful smoke GIF proves the full VHS dependency chain better than syntax validation alone.

## Fail early

Do not start product-specific authoring until required checks pass. Report the exact missing layer: executable, Playwright browser, VHS Chromium launch, ttyd, encoder, or codec.
