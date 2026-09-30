# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![버전](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![라이선스](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

Babtab Chrome 확장 프로그램을 위한 **로컬 중계 서버**입니다. AI 에이전트(Cursor / Pi / Claude Code / …)를 실제 Chrome에 연결해 줍니다.

MV3 확장 프로그램은 스스로 포트를 listen 할 수 없기 때문에, 이 작은 프로그램이 다리 역할을 합니다. `localhost`에서 동작하므로 트래픽이 컴퓨터 밖으로 나가지 않습니다. Relay는 요청과 결과를 메모리에서만 전달합니다(페이지 관찰 및 스크린샷 포함). 페이지 내용을 저장하지 않습니다.

## 권장 설치 방법 (0.2.0)

Node.js 20+가 필요합니다. Chrome 웹 스토어에서 확장 프로그램을 설치하고 Babtab의 **Add to Cursor**를 누르세요. Cursor에서 추가 / 활성화한 뒤 Chrome으로 돌아와 **Approve**를 누르세요. AI 도구가 Relay를 자동으로 실행하므로 터미널을 계속 열어 둘 필요가 없습니다. 다른 도구는 **Other AI tools / install with a command**에서 설정할 수 있습니다.

전환 전에 기존 수동 Relay를 종료하세요. 자세한 내용은 [최신 가이드](README.md)를 참조하세요. 아래는 수동 HTTP 연결용 절차입니다.

## 인증된 디바이스 연결로 업그레이드

Relay와 확장 프로그램을 함께 업데이트하고, 확장 프로그램을 새로고침한 뒤 Side Panel에서 다시 연결한 다음, 각 에이전트마다 2단계를 다시 실행하세요. 확장 프로그램이 새 `dev_v2_` 디바이스 ID를 생성합니다. 이전 ID에 연결된 토큰으로는 새 연결을 제어할 수 없습니다. Relay는 이제 명시적으로 `127.0.0.1`에만 바인딩됩니다. 원격 배포에서는 `HOST`를 직접 설정하고 TLS(`wss://` / `https://`)로 전송 중 인증 정보를 보호하세요.

## 연결 방식 (3가지 역할)

```text
AI 에이전트(Cursor / Pi …) ←→ Relay(로컬 :3000) ←→ Chrome 확장 프로그램(Side Panel 포함)
     MCP 설정으로 연결              전달 + 페어링만 담당            탭에서 실제로 동작하는 주체
```

## 빠른 시작 (clone도 install도 불필요)

### 1단계: Chrome 확장 프로그램 설치

`chrome://extensions` → **개발자 모드** 사용 → **압축해제된 확장 프로그램 로드** → `dist` 폴더 선택.

> Chrome 웹스토어에 게시되면 이 단계는 "스토어에서 설치"로 바뀝니다.

### 2단계: Relay 실행 (둘 중 하나, 결과는 동일)

```bash
# A. Node 20+가 있는 경우: 설치 없이 바로 실행
npx @babtab/relay

# B. Node를 설치하고 싶지 않은 경우: GitHub Releases에서 OS에 맞는 실행 파일 다운로드
./babtab-relay-darwin-arm64   # 예: macOS Apple Silicon
```

다음 메시지가 보이면 실행 성공입니다(기본 포트 `3000`):

```text
[babtab-relay] listening on http://127.0.0.1:3000
```

고급 사용법(포트 변경, 토큰 저장 위치 지정):

```bash
PORT=3001 npx @babtab/relay
BABTAB_TOKEN_FILE=~/.babtab/relay-tokens.json npx @babtab/relay
```

> `npx`는 설치가 아닙니다. "다운로드해서 한 번 실행"하는 것으로, 터미널을 닫으면 Relay가 종료됩니다.

### 3단계: 확장 프로그램을 Relay에 연결

1. Chrome에서 아무 웹사이트나 열고, 툴바의 Babtab 아이콘을 클릭해 **Side Panel** 열기
2. 1단계에서 포트만 확인(기본값 `3000`, Relay와 일치해야 함) — URL 입력 불필요
3. **"Save & Connect" (저장 후 연결)** 클릭

이 단계에서 내 Chrome이 Relay의 디바이스 하나로 등록됩니다.
(VPS의 원격 Relay를 쓰는 경우에도 같은 단계의 *Advanced: custom Relay URL*에서 설정할 수 있습니다.)

### 4단계: AI 에이전트를 Relay에 연결 (Cursor 예시)

1. Side Panel 같은 페이지에서 2단계 → **Cursor** 클릭 → 한 줄 명령어를 복사:

```bash
npx -y @babtab/relay setup --target cursor --relay-url http://127.0.0.1:3000 --device <내-디바이스-ID>
```

2. 아무 터미널에 붙여넣어 실행. 6자리 페어링 코드가 표시되면 Side Panel 상단 배너에서 **Approve(승인)** 클릭.
3. 완료: 명령어가 `~/.cursor/mcp.json`에 `babtab`을 자동으로 기록합니다
   (기존 항목은 유지, 이전 파일은 `.bak`으로 백업). Cursor에서 MCP를 새로고침하면
   (Settings → MCP) 패널에 `Controlled by: cursor`가 표시됩니다.

지원하는 `--target`: `cursor`, `claude-code`, `windsurf`, `copilot`,
`copilot-insiders`, `codex`, `pi`, `claude-desktop`, `antigravity`, `devin`,
`kimi`, `hermes`, `manual`. `npx -y @babtab/relay setup --help`에서 전체 명령어 확인 가능
(`--list-targets`는 하네스별 완성형 명령어를 한 번에 출력). 터미널을 쓸 수 없는 환경에서는
2단계의 *Pair & copy JSON manually*로 수동 붙여넣기로 돌아갈 수 있습니다.

3. 패널에 `Controlled by: cursor`가 표시되면 연결 완료.

동작 확인용 한 마디(에이전트에게 요청):

> browser_observe로 지금 Chrome에 열려 있는 탭을 확인한 뒤, 제목과 URL을 알려줘

## 자주 묻는 질문

- **"Save & Connect"을 눌러도 반응이 없어요.** 먼저 Relay 터미널에 `listening`이 표시되는지 확인하고, 1단계의 포트가 Relay 포트와 일치하는지 확인하세요.
- **페어링 코드가 만료됐어요.** 코드 유효 시간이 짧으니 setup 명령어를 다시 실행해 주세요.
- **토큰을 바꾸고 싶어요.** setup 명령어를 다시 실행하면 새 토큰이 발급되고 설정도 자동으로 다시 기록됩니다(이전 파일은 `.bak`으로 유지).
- **에이전트와 Chrome이 서로 다른 컴퓨터에 있어요.** (고급) Relay를 VPS에 올리고, Side Panel 1단계의 *Advanced: custom Relay URL*에 `wss://…`를 설정한 뒤 setup 명령어에 `--relay-url https://…`를 붙이면 됩니다. 과정은 동일합니다.

## 개인정보 보호

- `localhost`에서는 완전한 로컬 연결이라 패킷이 컴퓨터 밖으로 나가지 않습니다.
- Relay는 명령과 결과 전달만 하며, 페이지 내용을 파싱하거나 저장하지 않습니다.
- Side Panel에서 언제든 **Pause(일시정지) / Take Over(제어권 가져오기) / 연결 끊기**가 가능합니다. 최종 제어권은 항상 사람에게 있습니다.

## 개발자 안내

이 저장소에는 릴리스 산출물(난독화된 단일 번들 + 실행 파일)만 포함되며, 개발 소스는 포함되지 않습니다. Issue와 토론은 이 저장소에 직접 올려주세요.
