# Review and export

## Visual review

Use `ffprobe` to inspect duration, dimensions, and codec. Extract representative frames or a contact sheet with `ffmpeg`, then inspect the beginning, each major action, and the verified final state.

Scan visible text for likely sensitive material: home paths, email addresses, usernames, API keys, bearer tokens, JWTs, connection strings, `.env` values, private hosts, repository names, issue content, and browser tabs. Treat OCR as an aid, not proof of safety.

## MP4

```sh
ffmpeg -y -i input.webm -c:v libx264 -crf 22 -preset medium -pix_fmt yuv420p -movflags +faststart -an output.mp4
```

## GIF

```sh
ffmpeg -y -i input.webm -filter_complex "[0:v]fps=12,scale=800:-1:flags=lanczos,split[a][b];[a]palettegen=max_colors=128:stats_mode=diff[p];[b][p]paletteuse=dither=sierra2_4a" output.gif
```

If available, compare a gifsicle pass:

```sh
gifsicle -O3 --lossy=30 --colors 128 output.gif -o output.optimized.gif
```

Keep the smaller visually acceptable output. Prefer lower fps or width over extreme lossiness.

## Verification

- `ffprobe` reports valid metadata and nonzero duration.
- GIF is under the selected target, normally 8 MB.
- MP4 uses a broadly compatible pixel format.
- First and last frames are intentional.
- README path resolves correctly.
- Alt text says what the demo shows.
