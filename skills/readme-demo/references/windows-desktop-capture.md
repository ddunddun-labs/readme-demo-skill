# Windows native and Electron demos

Use this guide when an approved storyboard records a Windows native application or an Electron window. The goal is a reproducible adapter for one project, not a claim that every desktop UI can be automated identically.

## Separate control from capture

Treat the workflow as two adapters:

- **Control:** Windows UI Automation (UIA), with coordinate input only as a measured fallback.
- **Capture:** Windows Graphics Capture, `PrintWindow`, or visible-screen capture, selected after a smoke frame.

Keep launch, fixture setup, window placement, and accessibility setup outside the recorded interval when practical.

## UI Automation strategy

Prefer stable automation properties such as `AutomationId`, control type, and accessible name. Inspect the real UIA tree before authoring selectors; Electron, Win32, WPF, WinUI, and custom-rendered controls expose different patterns.

Choose an action from the patterns the element actually supports:

| Intent | Preferred UIA pattern | Possible fallback |
| --- | --- | --- |
| Activate a button or command | `InvokePattern` | `GetClickablePoint` plus pointer input |
| Enter text or a repository query | `ValuePattern` | Focus the element, then keyboard input |
| Select a list/tree item | `SelectionItemPattern` | Clickable point |
| Toggle a checkbox or switch | `TogglePattern` | Clickable point |
| Expand a menu or tree node | `ExpandCollapsePattern` | Clickable point |

PowerShell can probe a pattern without assuming it exists:

```powershell
Add-Type -AssemblyName UIAutomationClient
Add-Type -AssemblyName UIAutomationTypes

$pattern = $null
if ($element.TryGetCurrentPattern(
    [System.Windows.Automation.InvokePattern]::Pattern,
    [ref]$pattern
)) {
    ([System.Windows.Automation.InvokePattern]$pattern).Invoke()
}
```

For a field that exposes `ValuePattern`:

```powershell
$pattern = $null
if ($element.TryGetCurrentPattern(
    [System.Windows.Automation.ValuePattern]::Pattern,
    [ref]$pattern
)) {
    ([System.Windows.Automation.ValuePattern]$pattern).SetValue('owner/repository')
}
```

If the preferred pattern is unavailable, reacquire the element, obtain its current bounding rectangle or clickable point, verify that the point lies inside the target window, and only then use pointer input. Re-query geometry immediately before clicking because layout, DPI, and monitor placement may have changed.

## Capture method selection

| Method | Consider it when | Verify carefully |
| --- | --- | --- |
| Windows Graphics Capture (WGC) | Smooth continuous window capture, GPU-rendered or Electron content, or capture that should be less dependent on visible screen pixels | OS support, consent/picker or programmatic adapter, capture border behavior, window resizing, D3D/WinRT lifetime |
| `PrintWindow` | A conventional Win32-style window renders correctly into an off-screen device context | Black/stale frames, missing GPU surfaces, transparency and child-window rendering |
| `CopyFromScreen` / BitBlt-style visible capture | A simple adapter is useful and the target can remain fully visible and unobstructed | Occlusion, notifications, cursor, multi-monitor origins, DPI conversion, unrelated content |

WGC is often the best first candidate for continuous native or Electron recording, but it adds WinRT and graphics setup. `PrintWindow` can be convenient when the target supports it. Visible-screen capture is a reasonable fallback when foreground and privacy conditions can be controlled. Always decide from an inspected smoke frame rather than the API name alone.

## DPI and multiple monitors

- Choose the automation process DPI-awareness model before collecting coordinates and keep it stable for the run.
- Record the target monitor, its scale factor, the virtual-screen origin, and the target window rectangle.
- Label stored coordinates as logical units or physical pixels.
- Re-query the UIA rectangle after moving the window between monitors.
- Avoid negative-coordinate assumptions; monitors to the left or above the primary display may use negative virtual-screen coordinates.
- Fix the final capture size separately from the on-screen window size, and document any scaling step.

If coordinates disagree, pause and inspect the coordinate spaces rather than adding an unexplained offset.

## Foreground window and Z-order

Before coordinate input or visible-screen capture:

1. Restore the target if minimized.
2. Request foreground activation.
3. Wait briefly for the window manager and application to settle.
4. Verify the foreground HWND matches the intended target.
5. Reacquire the UIA element and its geometry.

Avoid leaving the application permanently topmost unless the approved scenario needs it. For a stable run, a dedicated virtual desktop or otherwise quiet desktop is often preferable. Ask the user not to interact with the mouse or keyboard during coordinate-based recording and provide an abort path. Avoid system-wide input blocking as the default because a failed script can leave the session difficult to control.

## Electron accessibility

Electron normally enables accessibility when assistive technology requests it. For an Electron project you control, a demo/test-only path may call this after the app is ready:

```javascript
app.whenReady().then(() => {
  app.setAccessibilitySupportEnabled(true)
})
```

Confirm that the expected accessible names, roles, and patterns appear in UIA. Accessibility tree rendering can affect performance, so avoid enabling it globally in production solely for demo automation. If the Electron UI already has stable end-to-end hooks, consider using them for state setup while keeping the recorded interaction truthful.

## WSL + Windows mixed workflow

When the agent runs in WSL:

1. Use `powershell.exe` or `pwsh.exe` for Windows process launch, UIA, HWND, DPI, foreground control, and capture.
2. Save the original capture to a Windows directory with a short, non-sensitive path.
3. Convert the result path with `wslpath`.
4. Use Linux ffmpeg/ffprobe for trimming, review frames, MP4, GIF, and size optimization.

Prefer transferring a source clip or frame sequence after capture instead of piping raw frames across the WSL boundary. Keep both Windows and WSL commands in the reproducible scenario and quote converted paths.

## Recording checklist

- Confirm the storyboard and final visible success state.
- Close or hide notifications, unrelated windows, personal paths, and account details.
- Place the app on the intended monitor and keep the DPI model unchanged.
- Pass one UIA action and one capture-frame smoke test.
- Verify foreground ownership before any coordinate fallback.
- Start capture after setup; hold the verified final state for 1–2 seconds.
- Inspect representative frames before encoding.
- Keep the original MP4 or source clip until the GIF passes review.

When a control or renderer cannot be automated reliably, describe the unsupported layer and propose a smaller truthful scenario rather than masking the failure with editing.
