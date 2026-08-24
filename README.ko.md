# README Demo Skill

[English](README.md) | **한국어**

![README Demo 아이콘](skills/readme-demo/assets/readme-demo-icon.png)

개발자 프로젝트의 README 데모를 설계하고, 녹화하고, 검토하고, 최적화해 문서에 삽입하는 Codex 스킬입니다.

일반적인 수동 화면 녹화 대신, 데모를 발표용 엔드투엔드 테스트처럼 다룹니다.

- 웹 애플리케이션은 Playwright 1.59+ Screencast로 녹화
- CLI와 TUI 화면은 VHS로 녹화
- Windows 네이티브와 Electron 앱은 UI Automation 기반 조작 지침 제공
- 검토, 합성, 내보내기는 FFmpeg 활용
- 녹화 해상도 고정 및 assertion 기반 성공 조건 확인
- 개인 경로, 이메일, 비밀 값, 무관한 화면 노출 검사
- 에이전트가 웹 앱을 조작하는 과정을 보여줄 때 브라우저·터미널 합성 패턴 선택 가능

## 설치

저장소를 clone한 뒤 `skills/readme-demo` 폴더를 로컬 Codex 스킬 경로에 복사합니다.

### macOS 및 Linux

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

설치 후 Codex를 다시 시작합니다. 다음과 같이 스킬을 명시적으로 요청할 수 있습니다.

```text
$readme-demo를 사용해서 이 프로젝트의 15초짜리 README 데모를 만들어줘.
```

README GIF나 MP4, 브라우저 사용 과정, 터미널 녹화, 릴리스 데모 또는 기능을 시각적으로 증명하는 결과물을 요청하면 Codex가 이 스킬을 자동으로 선택할 수도 있습니다.

## 요구 사항

모든 도구를 한꺼번에 설치하는 것이 아니라 녹화할 화면에 맞는 도구만 사용합니다.

- 웹 데모: Node.js, Playwright 1.59+, Playwright Chromium, FFmpeg
- 터미널 데모: VHS, ttyd, VHS Chromium, FFmpeg
- 브라우저·터미널 합성: 두 도구 모음 모두 필요
- Windows 네이티브/Electron 데모: PowerShell, UI Automation, 시험 프레임으로 검증한 캡처 방식, Windows 또는 WSL의 FFmpeg

선택한 작업 흐름에 맞춰 사전 점검과 실행·렌더링 smoke test를 수행하도록 구성되어 있습니다.

Windows 데스크톱 데모에서는 하나의 캡처 방식을 모든 앱에 적용하기보다 Windows Graphics Capture, `PrintWindow`, 화면 기반 캡처를 비교한 뒤 시험 프레임으로 선택합니다. UIA 패턴 fallback, 다중 창 합성, 모니터별 DPI, 전면 창 확인, Electron 접근성, Windows·WSL 혼합 인코딩 흐름도 함께 다룹니다. WinApp CLI를 사용할 수 있다면 프로젝트별 P/Invoke 코드를 작성하기 전에 선택 가능한 어댑터로 시험합니다.

## 저장소 구조

```text
skills/readme-demo/
├── SKILL.md
├── agents/openai.yaml
├── assets/
└── references/
```

## 출처

이 스킬은 MIT 라이선스로 공개된 [Conor Bronsdon의 `demo-gif-skill`](https://github.com/conorbronsdon/demo-gif-skill) 버전 1.0.0을 바탕으로 수정했습니다. 원본 저작권과 라이선스 고지는 [`references/upstream-notice.md`](skills/readme-demo/references/upstream-notice.md)에 보존되어 있습니다.

## 라이선스

[MIT](LICENSE)
