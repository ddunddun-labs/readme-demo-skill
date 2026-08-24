# Capture preflight

Run preflight after selecting the capture surfaces and before writing the final demo. Adapt commands to the host shell and project package manager. Do not require terminal dependencies for an ordinary browser-only demo.

## Dependency matrix

| Selected surface | Required checks |
| --- | --- |
| Browser only | Playwright package, Playwright Chromium launch, ffmpeg, ffprobe |
| Terminal only | vhs, ttyd, VHS Chromium render smoke test, ffmpeg, ffprobe |
| Browser + terminal composite | All browser and terminal checks |
| Windows native/Electron | Windows host and PowerShell, optional WinApp CLI, target process/window, UI Automation tree and control patterns, DPI/monitor state, capture smoke frame, chosen ffmpeg/ffprobe environment |
| Windows capture + WSL encoding | All Windows native checks, PowerShell invocation from WSL, path conversion, Linux ffmpeg and ffprobe |

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

## Windows native/Electron smoke checks

Run these checks from Windows PowerShell, or call PowerShell from WSL when using the mixed workflow:

```powershell
$PSVersionTable.PSVersion
Get-Command winapp -ErrorAction SilentlyContinue
Get-Command ffmpeg -ErrorAction SilentlyContinue
Get-Command ffprobe -ErrorAction SilentlyContinue
Get-Process | Select-Object ProcessName, Id, MainWindowTitle
```

WinApp CLI is optional. When available, consider using its UIA, screenshot, and recording commands before writing new P/Invoke helpers:

```powershell
winapp ui list-windows -a <process-or-title>
winapp ui inspect -w <HWND> --json
winapp ui screenshot -w <HWND>
```

If it is unavailable or does not support the scenario, continue with the PowerShell/UIA and capture-adapter checks below rather than treating the missing CLI as a blocker.

Before writing the full scenario:

1. Resolve the intended process, top-level window, and HWND without relying on a broad title substring when a stable identifier is available.
2. Inspect the UI Automation tree and record the patterns exposed by each intended control. A button may not provide `InvokePattern`; a text or search field may require `ValuePattern`.
3. Record the target monitor, effective DPI scale, window rectangle, and whether coordinates are logical or physical pixels. Keep the same DPI-awareness model throughout the run.
4. Bring the target window forward, then verify that it is the foreground window before any coordinate-based input.
5. Exercise one representative UIA action. If a coordinate fallback may be needed, test one clickable point before recording.
6. Capture and inspect one disposable frame with the proposed capture method. Look for a black frame, stale content, clipping, transparency loss, unintended borders, and occlusion by another window.
7. Confirm the selected ffmpeg environment can decode that frame or short source clip and write a disposable MP4.

For an Electron project you control, also confirm that its accessibility tree is available. If needed, enable accessibility support in a demo/test mode after Electron is ready; avoid making it a production default solely for recording.

### WSL and Windows path check

Keep UI control and capture on the Windows side. A practical mixed workflow is to save a source clip or frame sequence to a Windows path, convert it with `wslpath`, and run Linux ffmpeg afterward:

```sh
powershell.exe -NoProfile -File "$(wslpath -w scripts/demo/capture.ps1)"
source_path="$(wslpath 'C:\\Temp\\readme-demo\\source.mp4')"
ffprobe -v error "$source_path"
```

Choose the encoder side explicitly:

- **Windows ffmpeg:** check it with `Get-Command ffmpeg` and `Get-Command ffprobe` in PowerShell.
- **WSL ffmpeg:** check it with `command -v ffmpeg` and `command -v ffprobe` in WSL, then probe the converted Windows path.

Only one side needs to perform final encoding unless the scenario intentionally uses both. Prefer a file boundary over streaming high-rate raw frames between Windows and WSL. Verify quoting with a disposable path before the real recording.

## Fail early

Begin product-specific authoring after the checks relevant to the selected surface have passed. Report the exact missing layer—for example, executable, browser runtime, UI Automation pattern, coordinate mapping, foreground-window control, capture adapter, encoder, or codec—and suggest a practical fallback when one is available.
