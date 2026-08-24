# Windows native and Electron demos

Use this guide when an approved storyboard records a Windows native application or an Electron window. The goal is a reproducible adapter for one project, not a claim that every desktop UI can be automated identically.

## Separate control from capture

Treat the workflow as two adapters:

- **Control:** Windows UI Automation (UIA), optionally through WinApp CLI, with coordinate input only as a measured fallback.
- **Capture:** Windows Graphics Capture, `PrintWindow`, or visible-screen capture, selected after a smoke frame.

Keep launch, fixture setup, window placement, and accessibility setup outside the recorded interval when practical.

## Optional WinApp CLI adapter

When `winapp` is available, consider it before creating project-specific P/Invoke scripts. Its `ui` commands expose UIA inspection and actions, HWND targeting, screenshots, MP4 recording, JSON results, and bounded state waits:

```powershell
winapp ui list-windows -a Companion
winapp ui inspect -w <HWND> --json
winapp ui invoke <selector> -w <HWND>
winapp ui set-value <selector> "owner/repository" -w <HWND>
winapp ui wait-for <selector> -w <HWND> --value "Done" --timeout 5000
winapp ui record -w <HWND> --duration-sec 10 --fps 15 --output demo.mp4
```

Prefer its pattern-based verbs for non-visual setup and assertions. Its pointer and keyboard injection verbs still need an unlocked interactive desktop and the intended foreground window. Treat a nonzero exit or structured JSON error as a failed step, and confirm visible state after an injected action.

WinApp CLI is a convenience adapter, not a requirement. Keep the UIA and capture selection guidance below as the fallback and as the basis for diagnosing tool-specific failures.

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

## Verify postconditions

After each meaningful action, wait for a visible or accessible state rather than assuming the call succeeded. Prefer a UIA event when the provider raises a reliable one, with a bounded property/element poll as a practical fallback because some providers omit change events.

```powershell
winapp ui wait-for StatusText -w <HWND> --property Name --value "Ready" --timeout 5000
```

For a custom PowerShell adapter, poll a specific property or element at a short interval until a fixed deadline. Report the expected state and last observed state on timeout. Avoid an unbounded loop or a long presentation sleep in place of verification.

## Capture method selection

| Method | Consider it when | Verify carefully |
| --- | --- | --- |
| Windows Graphics Capture (WGC) | Smooth continuous window capture, GPU-rendered or Electron content, or capture that should be less dependent on visible screen pixels | OS support, consent/picker or programmatic adapter, capture border behavior, window resizing, D3D/WinRT lifetime |
| `PrintWindow` | A conventional Win32-style window renders correctly into an off-screen device context | Hangs, black/stale frames, missing GPU surfaces, transparency and child-window rendering |
| `CopyFromScreen` / BitBlt-style visible capture | A simple adapter is useful and the target can remain fully visible and unobstructed | Occlusion, notifications, cursor, multi-monitor origins, DPI conversion, unrelated content |

WGC is often the best first candidate for continuous native or Electron recording, but it adds WinRT and graphics setup. `PrintWindow` can be convenient when the target supports it; invoke it behind a bounded timeout so an unresponsive target does not stall the workflow. Discard black, stale, or incomplete output and try WGC or visible-screen capture instead. Visible-screen capture is a reasonable fallback when foreground and privacy conditions can be controlled. Decide from an inspected smoke frame rather than the API name alone.

## Multiple application windows

When the storyboard shows two separate applications, first consider capturing each window independently and composing the clips on a fixed canvas. This reduces sensitivity to which HWND is foreground and makes each source easier to inspect for privacy and rendering failures.

For independent capture:

- Fix each source window's capture dimensions.
- Define the final canvas, placement, gap, and background explicitly.
- Synchronize the clips to visible actions or markers.
- Hold or trim each source deliberately when one clip is shorter.
- Inspect both panes at the beginning, transition points, and final state.

Use a shared screen capture when the spatial handoff between applications is itself part of the demonstration. In that case, fix both window rectangles and their Z-order, verify the intended foreground HWND before each injected action, and inspect a smoke frame for partial occlusion of either target. A foreground check covers only one window; it does not prove that the second window is unobstructed.

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

For a third-party Electron app that cannot be modified, Chromium's `--force-renderer-accessibility=complete` launch switch may expose a fuller tree:

```powershell
& "C:\\Path\\To\\App.exe" --force-renderer-accessibility=complete
```

Treat this as a compatibility experiment rather than an assumed Electron contract. Fully exit existing app and helper processes first, launch with the switch, and compare the UIA tree before relying on it. Single-instance launchers may forward or ignore arguments, and app or Chromium updates may change behavior; record the tested app version and re-check after upgrades.

## WSL + Windows mixed workflow

When the agent runs in WSL:

1. Use `powershell.exe` or `pwsh.exe` for Windows process launch, UIA, HWND, DPI, foreground control, and capture.
2. Save the original capture to a Windows directory with a short, non-sensitive path.
3. Convert the result path with `wslpath`.
4. Use Linux ffmpeg/ffprobe for trimming, review frames, MP4, GIF, and size optimization.

Prefer transferring a source clip or frame sequence after capture instead of piping raw frames across the WSL boundary. Keep both Windows and WSL commands in the reproducible scenario and quote converted paths.

## Windows-specific handoff

Follow the shared review and export workflow in `references/review-and-export.md`. In addition, report the target HWNDs, monitor/DPI assumptions, UIA or input fallback used, capture mode selected by the smoke test, encoder side (Windows or WSL), and any third-party Electron launch flag.

When a control or renderer cannot be automated reliably, describe the unsupported layer and propose a smaller truthful scenario rather than masking the failure with editing.

## References

- WinApp CLI UI Automation: https://learn.microsoft.com/en-us/windows/apps/dev-tools/winapp-cli/ui-automation
- Microsoft UI Automation events: https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-eventsforclients
- Electron accessibility: https://www.electronjs.org/docs/latest/tutorial/accessibility
- Chromium accessibility overview: https://chromium.googlesource.com/chromium/src/+/main/docs/accessibility/overview.md
