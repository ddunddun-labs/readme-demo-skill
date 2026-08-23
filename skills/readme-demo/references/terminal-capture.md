# Terminal capture

Prefer Charmbracelet VHS for CLI, TUI, and library examples because `.tape` files are reproducible.

Run the complete dependency and render smoke test in `preflight.md` first. VHS needs more than the `vhs` executable: verify `ttyd`, `ffmpeg`, and the Chromium runtime that VHS launches.

```text
Output docs/demo.gif
Set Shell "bash"
Set Width 1200
Set Height 600
Set FontSize 18
Set Framerate 12

Type "tool-name --help"
Enter
Sleep 2s
```

- Hide setup and dependency installation from capture.
- Use disposable fixtures and deterministic output.
- Never type real tokens, usernames, home paths, private hosts, or production identifiers.
- Confirm `vhs`, `ttyd`, and `ffmpeg` before rendering.
- Validate syntax and inspect the rendered result.
- Prefer a wrapper script for complex shell quoting.
- Use asciinema plus `agg` only when VHS is unavailable and note reduced reproducibility.
- When compositing with a longer browser recording, play the terminal animation once and hold its final frame. Never let an embedded GIF's infinite-loop metadata create repeated terminal actions.

Documentation: https://github.com/charmbracelet/vhs
