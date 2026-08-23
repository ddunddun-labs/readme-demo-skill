# CLI or agent driving a web app

Use this reference only when the approved story intentionally shows a CLI or coding agent causing a visible change in a web application. Do not load or apply it to ordinary browser-only demos or independent terminal demos.

This pattern has two synchronized surfaces:

- the terminal shows the human issuing a natural-language request through a CLI or agent;
- the browser shows the real application state changing as a consequence of the actual MCP, API, or agent operation.

The terminal is not decorative narration. It must represent the real user intent, while the browser must show and verify the real result.

## Composition

Use one explicit final canvas, normally 1280×720. Record the browser at that size. Scale and place the terminal inside a deliberate region or compose the two sources side by side.

## Hold the terminal's final frame

Animated GIFs commonly contain infinite-loop metadata. Ignore that loop, play the terminal animation once, and repeat only its final decoded frame until the browser video ends.

```sh
ffmpeg -y \
  -i browser.mp4 \
  -ignore_loop 1 -i terminal.gif \
  -filter_complex \
  "[0:v]scale=1280:720,setsar=1[base]; \
   [1:v]scale=480:-1,setsar=1[term]; \
   [base][term]overlay=x=W-w-32:y=H-h-32:eof_action=repeat[v]" \
  -map "[v]" -an \
  -c:v libx264 -crf 22 -preset medium \
  -pix_fmt yuv420p -movflags +faststart \
  composite.mp4
```

Key points:

- `-ignore_loop 1` decodes one terminal-GIF cycle instead of obeying infinite-loop metadata.
- `eof_action=repeat` holds the terminal stream's final frame after it ends.
- Do not add `shortest=1`; that would end composition when the short terminal input ends.
- The main browser input determines total duration. Verify final duration with `ffprobe`.
- If the installed ffmpeg build behaves differently, first convert the GIF to a one-shot intermediate video and add `tpad=stop_mode=clone` for an explicit hold.

## Side-by-side alternative

For equal panels, scale and pad both inputs to their assigned rectangles and combine with `hstack`. Extend the shorter terminal source with `tpad=stop_mode=clone` before stacking. Never use `-stream_loop -1` for a command sequence unless repetition is intentionally part of the story.

## Displayed CLI input plus real background operation

In this specific CLI-to-web-app pattern, it is acceptable to show a natural-language request being typed in the terminal while a separate automation helper invokes the actual MCP operation. Keep it truthful and reproducible:

1. Define one prompt string and reuse it for both visible typing and the MCP call when possible.
2. Mark the visible terminal input in source comments as presentation input when it is not itself the invocation channel.
3. Trigger the real MCP call at a defined timeline event.
4. Wait for and assert the actual resulting state before ending capture.
5. Do not imply that an unconnected terminal or browser field directly submitted the request.

The visible CLI action explains the human request; the real background call proves the CLI/agent-to-web-app behavior. Both must be preserved in the reproducible scenario.
