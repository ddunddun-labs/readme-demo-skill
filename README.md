# README Demo Skill

![README Demo icon](skills/readme-demo/assets/readme-demo-icon.png)

A Codex skill for designing, recording, reviewing, optimizing, and embedding reproducible README demos for developer projects.

It treats a demo as a presentation-oriented end-to-end test rather than a manual screen recording:

- Playwright 1.59+ Screencast for web applications
- VHS for CLI and TUI surfaces
- FFmpeg for review, composition, and export
- Fixed capture dimensions and assertion-based success criteria
- Privacy checks for paths, email addresses, secrets, and unrelated content
- An optional browser-and-terminal composition pattern for agent-driven workflows

## Install

Clone the repository and copy the skill directory into your local Codex skills directory.

### macOS and Linux

```bash
git clone https://github.com/ddunddun-labs/readme-demo-skill.git
mkdir -p ~/.codex/skills
cp -R readme-demo-skill/skills/readme-demo ~/.codex/skills/readme-demo
```

### Windows PowerShell

```powershell
git clone https://github.com/ddunddun-labs/readme-demo-skill.git
New-Item -ItemType Directory -Force "$HOME\.codex\skills" | Out-Null
Copy-Item -Recurse readme-demo-skill\skills\readme-demo "$HOME\.codex\skills\readme-demo"
```

Restart Codex after installation. Then request it explicitly:

```text
Use $readme-demo to create a 15-second README demo of this project.
```

Codex can also select the skill automatically when a request asks for a README GIF or MP4, browser walkthrough, terminal recording, release demo, or other visual proof of a feature.

## Requirements

Dependencies are selected by capture surface rather than installed all at once:

- Web demos: Node.js, Playwright 1.59+, Playwright Chromium, and FFmpeg
- Terminal demos: VHS, ttyd, VHS Chromium, and FFmpeg
- Browser-and-terminal composites: both toolchains

The skill includes preflight checks and launch/render smoke tests for the selected workflow.

## Repository layout

```text
skills/readme-demo/
├── SKILL.md
├── agents/openai.yaml
├── assets/
└── references/
```

## Attribution

This skill is adapted from [Conor Bronsdon's `demo-gif-skill`](https://github.com/conorbronsdon/demo-gif-skill), version 1.0.0, under the MIT License. The upstream copyright and license notice are retained in [`references/upstream-notice.md`](skills/readme-demo/references/upstream-notice.md).

## License

[MIT](LICENSE)
