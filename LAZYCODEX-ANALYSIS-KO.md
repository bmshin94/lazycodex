# LazyCodex 전수조사 & 수익화 분석 리포트

> 작성일: 2026-10-01
> 분석 대상 커밋: `2bfa138` (main) / 브랜치 `claude/dazzling-babbage-5gcn61`
> 분석 도구: Claude Code (카리나 페르소나)

---

## 📌 관련 GitHub 주소 모음

| 구분 | URL |
|---|---|
| **이 리포지터리 (내 것)** | https://github.com/bmshin94/lazycodex |
| **원본 리포지터리** | https://github.com/code-yeongyu/lazycodex |
| **실제 엔진 (OmO)** | https://github.com/code-yeongyu/oh-my-openagent |
| **Codex 구현체 위치** | https://github.com/code-yeongyu/oh-my-openagent/tree/dev/packages/omo-codex |
| **npm 패키지** | https://www.npmjs.com/package/lazycodex-ai |
| **공식 홈페이지** | https://lazycodex.ai |
| **공식 문서** | https://lazycodex.ai/docs |
| **이슈 트래커** | https://github.com/code-yeongyu/lazycodex/issues |
| **Discord (LazyCodex)** | https://discord.gg/6ztZB9jvWq |
| **Discord (building-in-public)** | https://discord.gg/PUwSMR9XNk |
| **제작사 (Sisyphus Labs)** | https://sisyphuslabs.ai |
| **참고: LazyVim (이름 영감)** | https://github.com/LazyVim/LazyVim |
| **참고: oh-my-codex (개념 영감)** | https://github.com/Yeachan-Heo/oh-my-codex |

---

## 1. 한 줄 정의

> **LazyCodex = OpenAI Codex를 "AI 개발팀"으로 바꿔주는 설치형 에이전트 하네스(harness) 배포판**

`LazyVim`이 Neovim을 바로 쓸 수 있게 세팅해주듯, LazyCodex는 Codex를 바로 쓸 수 있게 세팅해준다.

---

## 2. 리포지터리 기본 정보

| 항목 | 내용 |
|---|---|
| npm 패키지명 | `lazycodex-ai` v0.2.2 |
| 플러그인 패키지명 | `@sisyphuslabs/omo-codex-plugin` v5.0.0-beta.72 |
| 마켓플레이스 | `sisyphuslabs` |
| 플러그인 이름 | `omo` |
| 라이선스 | MIT |
| 저자 | Yeongyu Kim (Sisyphus Labs) |
| 유지보수 | Jobdori (AI 어시스턴트) |

### ⚠️ 중요 주의사항 2가지

1. **`src/` 폴더가 비어있다.** `.gitmodules`에 `oh-my-openagent` 서브모듈로 등록만 되어 있다.
   실제 엔진 코드를 보려면:
   ```bash
   git submodule update --init --recursive
   ```

2. **이 리포는 "생성된 배포 미러"다.**
   `.github/workflows/pr-source-guidance.yml`이 들어오는 모든 PR을
   `pull_request_target` 이벤트에서 **자동으로 닫는다.**
   (원본 소스에서 매 릴리즈마다 재생성되므로 PR 머지가 구조적으로 불가능)
   → 포크인 `bmshin94/lazycodex`에서도 이 워크플로가 그대로 동작한다.

---

## 3. 폴더 구조 해부

```
lazycodex/
├── bin/lazycodex-ai.js      → 28줄짜리 설치 별칭 CLI
├── .agents/plugins/
│   └── marketplace.json     → Codex 마켓플레이스 등록 (sisyphuslabs)
├── plugins/omo/             → ★ 실제 Codex 플러그인 본체
│   ├── .codex-plugin/plugin.json  → 플러그인 매니페스트
│   ├── .mcp.json            → MCP 서버 4개 정의
│   ├── model-catalog.json   → 모델 라우팅 카탈로그
│   ├── skills/      (26개)  → 스킬 플레이북 (SKILL.md)
│   ├── hooks/       (21개)  → 생명주기 훅 JSON
│   ├── components/  (18개)  → TS 구현체 + dist 번들
│   └── shared/              → 통합 설정 로더 + 마이그레이션
├── packages/web/            → lazycodex.ai 공식 사이트 (Next.js 16)
├── test/            (6개)   → 문서/바이너리 계약 테스트
├── .omo/                    → 과거 작업 증거(스크린샷/diff) 아카이브
├── .github/workflows/ (5개) → CI/CD
└── CLAUDE.md                → 카리나 페르소나 정의 (직접 추가한 것)
```

### 3-1. `bin/lazycodex-ai.js` — 사실 "껍데기"

이 파일이 하는 일은 단 하나, 명령어 변환이다.

```
npx lazycodex-ai install
  ↓ 그대로 변환 ↓
npx --yes --package oh-my-openagent omo install --platform=codex
```

즉 **LazyCodex 자체는 설치 명령어 단축 별칭**이고, 실제 기능은 전부
`plugins/omo/`와 OmO 패키지에서 나온다. README가 스스로를
"thin distribution layer(얇은 배포 레이어)"라고 부르는 이유.

`--dry-run` 플래그를 주면 실행 없이 변환된 명령어만 출력한다.

### 3-2. 스킬 26개 전체 목록

| 스킬 | 역할 |
|---|---|
| `ulw-plan` | **Prometheus 플래너.** 제품 코드 절대 안 건드리고 계획서만 작성 |
| `ulw-execute` | 계획서를 Boulder 상태관리로 실행 |
| `ulw-loop` | 자기참조 루프 (ultrawork 500회 / 일반 100회 상한) |
| `ulw-research` | 클레임 그래프 게이팅 + 인용 기반 심층 리서치 |
| `ultrawork` | `ulw`/`ultrawork` 키워드 감지 시 강제 발동 모드 |
| `init-deep` | 디렉터리 복잡도 점수화 → 계층형 `AGENTS.md` 생성 |
| `programming` | Python/Rust/TypeScript/Go 엄격 규율 |
| `frontend` | "Linear·Stripe·Supabase 수준" UI 요구 |
| `visual-qa` | 스크린샷 증거 기반 시각 QA |
| `debugging` | 가설 주도 디버깅 + 실패 테스트로 고정 |
| `refactor` | 분해 전략 가이드 |
| `review-work` | 게이트 리뷰어 1명만 띄움 (패널 금지) |
| `remove-ai-slops` | "AI 티 나는 코드" 동작 보존 제거 |
| `git-master` | atomic 커밋, rebase, squash, blame, bisect, reflog |
| `lsp` / `lsp-setup` | 언어서버 진단·정의이동·참조찾기·리네임 |
| `ast-grep` | 25개 언어 AST 구조 검색/치환 |
| `teammode` | 이름 붙인 팀 병렬 실행 (durable state) |
| `data-scientist` | DuckDB/Polars 데이터 분석 |
| `ultimate-browsing` | WAF 우회 + 실브라우저 자동화 |
| `coding-agent-sessions` | 과거 에이전트 세션 로그 복구 |
| `rules` | AGENTS.md·룰파일 자동 주입 |
| `comment-checker` | 수정 후 주석 품질 피드백 |
| `lcx-doctor` | 설치 건강 진단 (PASS/WARN/FAIL) |
| `lcx-report-bug` | 버그 리포트 라우팅 |
| `lcx-contribute-bug-fix` | 검증된 버그 수정 기여 |

### 3-3. 훅 21개 — 핵심 차별점

| 타이밍 | 개수 | 하는 일 |
|---|---|---|
| `SessionStart` | 4 | 프로젝트 룰 로딩 · 텔레메트리 · 자동업데이트 · 부트스트랩 |
| `UserPromptSubmit` | 3 | 룰 재주입 · **ultrawork 트리거 감지** · ulw-loop 조종 |
| `PreToolUse` | 3 | Git Bash MCP 추천 · 목표예산 무제한화 · spawn 가드 |
| `PostToolUse` | 5 | **파일 수정 즉시 LSP 진단 + 주석 검사** · 룰 매칭 · 스레드 제목 검사 |
| `PostCompact` | 3 | 컨텍스트 압축 후 캐시 리셋 |
| `Stop` | 2 | **종료 시도를 가로채 "미완료면 계속" 강제** |
| `SubagentStop` | 1 | 워커 증거 산출물 실존 검증 |

**가장 중요한 설계:** `Stop` 훅 + `SubagentStop` 훅 조합으로
"AI의 조기 완료 선언"을 구조적으로 차단한다.

훅 구조 예시 (모든 훅이 동일 형태):
```json
{
  "type": "command",
  "command": "node \"${PLUGIN_ROOT}/components/rules/dist/cli.js\" hook session-start",
  "timeout": 10,
  "statusMessage": "(OmO 5.0.0-beta.72) Loading Project Rules",
  "commandWindows": "powershell ... node-dispatch.ps1 ..."
}
```
→ 훅은 단순히 **CLI 명령어를 실행**하는 구조라서, 언어 선택이 자유롭다.

### 3-4. MCP 서버 4개

| 이름 | 종류 | 주소/경로 | 인증 |
|---|---|---|---|
| `grep_app` | 원격 | `https://mcp.grep.app` | 불필요 |
| `context7` | 원격 | `https://mcp.context7.com/mcp` | 불필요 |
| `git_bash` | 로컬 stdio | `components/git-bash-mcp/dist/cli.js` | 불필요 |
| `lsp` | 로컬 stdio | `components/lsp-daemon/dist/cli.js` | 불필요 |

> AST 검색은 MCP가 아니라 `ast-grep` **스킬**로 제공된다.

### 3-5. 서브에이전트 12명 (`~/.codex/agents/*.toml`)

| 에이전트 | 모델 / 추론 수준 | 역할 | 쓰기권한 |
|---|---|---|---|
| `explorer` | gpt-6-astra / low, fast tier | 코드 위치 탐색 전문 | 읽기전용 |
| `librarian` | — | 문서·지식 조사 | 읽기전용 |
| `plan` | — | 계획 수립 | 읽기전용 |
| `metis` | — | 계획 검토 | 읽기전용 |
| `momus` | — | 비판적 검증 | 읽기전용 |
| `lazycodex-worker-low` | gpt-6-astra / high | 소규모 변경 (단일 파일) | 쓰기 |
| `lazycodex-worker-medium` | gpt-6-astra / — | 중규모 기능 (몇 개 파일) | 쓰기 |
| `lazycodex-worker-high` | gpt-6-astra / medium | 대규모 (신규 모듈, 크로스모듈 리팩터) | 쓰기 |
| `lazycodex-code-reviewer` | gpt-6-astra / medium | 코드품질 리뷰 | 읽기전용 |
| `lazycodex-qa-executor` | gpt-6-astra / high | 실제 시나리오 QA 실행 | 읽기전용 |
| `lazycodex-gate-reviewer` | gpt-6-astra / low | 최종 게이트 (APPROVE/REJECT) | 읽기전용 |
| `lazycodex-clone-fidelity-reviewer` | gpt-6-astra / high | 디자인이 "가짜"인지 검증 | 읽기전용 |

### 3-6. 모델 라우팅 (`model-catalog.json`)

현재 기본 프로필 (버전 `2026-09-08.gpt-6-astra-600k-high`):
```json
{
  "model": "gpt-6-astra",
  "model_context_window": 600000,
  "model_reasoning_effort": "high",
  "plan_mode_reasoning_effort": "xhigh"
}
```

레거시 프로필(`gpt-5.5` 400k/1M/272k, `gpt-5.6-sol` 650k)은 자동 업그레이드되고,
카탈로그 밖의 사용자 설정 모델은 **보존**된다.

README에 설명된 라우팅 철학 (= quota discipline):
- `quick` → `gpt-5.4-mini` (작은 편집)
- `ultrabrain` → 고추론 GPT 모델 (어려운 로직)
- agentic coding → Codex 튜닝 모델 (`gpt-5.3-codex` 등)

### 3-7. `packages/web/` — 공식 홈페이지

| 항목 | 값 |
|---|---|
| 프레임워크 | Next.js 16.2.9 |
| UI | React 19.2.7 + Tailwind CSS 4.3.1 |
| 언어 | TypeScript 5.9.3 |
| 린터 | Biome 2.5.0 |
| 배포 | Cloudflare Workers (`@opennextjs/cloudflare` 1.19.11) |
| 테스트 | Playwright 1.61.0 (e2e 9개) + Lighthouse 13.4.0 |
| 패키지 매니저 | pnpm 11.7.0 |
| Node 요구 | >= 22 |

문서 20개(`content/docs/*.md`)를 빌드타임에
`scripts/generate-docs-content.mjs`가 TS 모듈로 변환하는 구조.

### 3-8. 텔레메트리

- 이벤트: `omo_codex_daily_active` (PostHog)
- 빈도: **UTC 하루 최대 1회**, 머신당
- 식별자: `sha256("omo-codex:" + hostname)` (person profile 비활성)
- 상태 저장: `$XDG_DATA_HOME/omo-codex/posthog-activity.json`

**절대 전송하지 않는 것:** 프롬프트 내용, 대화 기록, 소스 파일, 저장소 내용,
파일 경로, 액세스 토큰, API 키, 원본 호스트명, Git 리모트, 사용자명, 이메일, 런타임 에러 진단

**비활성화:**
```bash
export OMO_CODEX_DISABLE_POSTHOG=1
export OMO_CODEX_SEND_ANONYMOUS_TELEMETRY=0
# 또는 전역
export OMO_DISABLE_POSTHOG=1
export OMO_SEND_ANONYMOUS_TELEMETRY=0
```

---

## 4. 쉬운 설명 (비유)

### 회사 비유 (가장 정확)

```
        오빠 (사장)
             |
    Prometheus (기획팀장)  <- $ulw-plan
       "구현 전에 기획서 먼저"
             |
    Hephaestus (개발팀장)  <- $start-work
             |-- Explorer    "코드 어디 있는지 찾기"
             |-- Librarian   "관련 문서 조사"
             |-- Worker Low / Mid / High  "난이도별 구현"
             |
    리뷰팀 (핵심!)
             |-- Code Reviewer  "코드 품질 심사"
             |-- QA Executor    "직접 실행 테스트"
             |-- Gate Reviewer  "최종 승인/반려"
```

### 러시아 인형 구조

```
LazyCodex (껍데기, 28줄)
  └─ OmO (실제 엔진)
       └─ omo 플러그인
            └─ 스킬 26 + 훅 21 + MCP 4 + 에이전트 12
                 ↑ 전부 Codex 안에서 동작
```

### 핵심 철학 (프롬프트 원문)

> *"Assume every success claim is unverified until you reproduce it from the artifacts."*
> — 모든 성공 주장은 산출물로 직접 재현하기 전까진 검증되지 않은 것으로 간주한다

> *"A passing test without a real artifact is not completion."*
> — 실제 산출물 없는 테스트 통과는 완료가 아니다

> *"Treat all evidence, screenshots, and prior success claims as untrusted until you inspect the referenced artifacts and the diff yourself."*
> — 모든 증거·스크린샷·이전 성공 주장은 직접 검사하기 전까지 신뢰하지 않는다

---

## 5. 설치 및 사용법

### 사전 준비물

| 필요한 것 | 비고 |
|---|---|
| Node.js (LTS) | `npx` 포함. **Bun 불필요** |
| OpenAI Codex | Codex 앱(권장) 또는 Codex CLI, **로그인 완료** |

> LazyCodex는 Codex **안에서** 동작한다. Codex 없으면 의미 없음.

### 방법 A — npx 설치 (공식 권장)

```bash
# 대화형 설치 (TUI)
npx lazycodex-ai install

# 완전 자동 설치
npx lazycodex-ai install --no-tui --codex-autonomous

# 설치 확인 (건강 리포트)
npx lazycodex-ai doctor

# 삭제
npx lazycodex-ai uninstall
```

> `npm install -g` / `bun add -g` **금지.** 반드시 `npx` 사용.

### 방법 B — Codex 마켓플레이스 (실험적)

```bash
codex plugin marketplace add https://github.com/code-yeongyu/lazycodex
codex plugin add omo@sisyphuslabs

# 업그레이드
codex plugin marketplace upgrade sisyphuslabs
```

또는 Codex 안에서 `/plugins` → **Add Marketplace** 탭 → 위 URL 입력 → `omo` 설치

### 설치 후 첫 실행 플로우

```
1. Codex 재시작
   → 시작 리뷰에서 "omo 훅 승인" 요청 → 승인
     (승인 전엔 훅이 절대 실행되지 않음 = 안전설계)

2. 부트스트랩 백그라운드 실행
   "LazyCodex bootstrap running in background — restart the session when it completes"
   → 완료되면 세션 재시작

3. 입력창에 $ 입력 → 스킬 목록 표시되면 성공

4. 첫 명령어: $init-deep
```

### 4대 명령어

```bash
# 1단계: 프로젝트 기억 생성 (처음 한 번)
$init-deep

# 2단계: 계획 수립 (제품 코드 안 건드림)
$ulw-plan "로그인에 소셜 로그인 추가"
  → plans/<slug>.md 생성

# 3단계: 계획 실행
$start-work [plan-name] [--worktree <path>]
  → 완료 시 "ORCHESTRATION COMPLETE" 출력

# 4단계: 검증까지 자동 반복
$ulw-loop "작업내용" [--completion-promise=TEXT] [--strategy=reset|continue]
  → ultrawork 모드 500회 / 일반 100회 상한
```

### 숨은 꿀팁

프롬프트 끝에 `ultrawork` 또는 `ulw`를 붙이면 `UserPromptSubmit` 훅이 감지해
울트라워크 모드가 자동 발동한다. 발동 시 첫 줄에 `ULTRAWORK MODE ENABLED!` 출력.

```
"결제 모듈 리팩터링해줘 ultrawork"
```

### Windows 주의사항

- **Git Bash 필수.** 없으면 설치기가 `winget install --id Git.Git -e --source winget` 자동 실행
- `where bash`가 `C:\Windows\System32\bash.exe`를 보여주면 그건 WSL 런처 → 무시됨.
  진짜 Git Bash 경로를 `OMO_CODEX_GIT_BASH_PATH`에 지정
- 마켓플레이스 설치 시 Node 없으면 `%USERPROFILE%\.codex\runtime\node\`에 자동 프로비저닝
- 부트스트랩 로그: `%USERPROFILE%\.codex\plugins\data\omo-sisyphuslabs\bootstrap\ps-bootstrap.log`

### 업그레이드마다 반복되는 현상

훅이 **Modified**로 표시된다. 플러그인 파일이 변경되어 이전 신뢰 해시가
맞지 않기 때문이며 **정상**이다. 재승인하면 다음 세션에서 새 버전 부트스트랩이 돌아간다.

---

## 6. 플러그인? 스킬? MCP? → 전부 다

`plugins/omo/.codex-plugin/plugin.json` 매니페스트가 답을 준다:

```json
{
  "name": "omo",
  "version": "5.0.0-beta.72",
  "skills": "./skills/",
  "hooks": [ /* 21개 JSON */ ],
  "mcpServers": "./.mcp.json",
  "interface": {
    "capabilities": ["Hooks", "MCP Tools", "Code Intelligence",
                     "Workflow", "Context Injection"],
    "brandColor": "#7C3AED"
  }
}
```

| 용어 | 여기서의 의미 |
|---|---|
| **플러그인** | 가장 큰 단위. `omo` 플러그인 = 전체 패키지 |
| **스킬** | 플러그인 안의 지식·절차서. `$이름`으로 호출 |
| **MCP** | 플러그인 안의 도구 연결 통로 (4개) |
| **훅** | **진짜 차별점.** 자동 개입 로직 (21개) |

> **결론: "MCP 서버와 스킬을 번들로 포함한, 훅 중심의 Codex 플러그인"**

---

## 7. API 토큰 필요 여부

> 공식 문서(`installation.md`) 원문:
> *"Auth targets **Codex itself**, not LazyCodex. There is **no separate LazyCodex login command**."*

| 대상 | 토큰 필요? | 설명 |
|---|---|---|
| LazyCodex 설치 | 불필요 | 로그인 명령어 자체가 없음 |
| **Codex 로그인** | **필수** | ChatGPT 구독 또는 OpenAI API 키 |
| `context7` MCP | 불필요 | 공개 원격 |
| `grep_app` MCP | 불필요 | 공개 원격 |
| `git_bash` MCP | 불필요 | 로컬 stdio 번들 |
| `lsp` MCP | 불필요 | 로컬 stdio 번들 |
| 텔레메트리 | 불필요 | 익명, 비활성화 가능 |

### Codex 인증 2가지 경로

```
방법 A: ChatGPT 구독 (Plus/Pro/Business)
  → 구독료에 포함, 추가 과금 없음
  → 설치기가 구독을 자동 감지해 모델 라우팅 설정
  → 권장 (예측 가능한 비용)

방법 B: OpenAI API 키 (sk-...)
  → 환경변수로 공급, 토큰당 과금
  → LazyCodex는 토큰 소비량이 크므로 비용 주의
```

### 보안 수칙 (문서 명시)

- 모든 `*_API_KEY`와 OAuth 자격증명은 비밀정보 — 로그/커밋 금지
- 키는 **환경변수**로만 공급
- *"Do not fabricate provider keys"* — 가짜 키 생성 금지

---

## 8. AI 에이전트 구축에 도움이 되는가 → 매우 그렇다

### 바로 차용 가능한 설계 패턴 7개

| # | 패턴 | 가치 | 핵심 |
|---|---|---|---|
| 1 | **훅 기반 생명주기** | ★★★★★ | 7개 이벤트에 자동 개입. 특히 `Stop` 훅으로 조기 종료 방지 |
| 2 | **증거 기반 완료 검증** | ★★★★★ | `SubagentStop`에서 증거 파일 실존/비어있음 검사 |
| 3 | **역할 기반 서브에이전트 분리** | ★★★★★ | 읽기전용 vs 쓰기권한 분리, 역할별 추론 수준 차등 |
| 4 | **비용 최적화 모델 라우팅** | ★★★★ | 난이도 → 모델 자동 매핑 (quota discipline) |
| 5 | **"불신" 기본값 프롬프팅** | ★★★★★ | 에이전트끼리도 서로 신뢰하지 않게 설계 |
| 6 | **스킬 라우터 패턴** | ★★★★ | "참조를 먼저 선언" → 컨텍스트 절약 + 책임 추적 |
| 7 | **negative triggering** | ★★★★ | description에 "언제 발동 안 하는지"를 더 길게 명시 |

### 패턴 6 원문 예시 (`frontend/SKILL.md`)

> *"This file is a **router, not a rulebook**. Before touching any file, name the references the request routes to and the one reason each is needed, then read exactly those. ... a reference you named but never opened is a gap you still owe."*

### 패턴 7 원문 예시 (`ulw-plan/SKILL.md`)

> *"ACTIVATES ONLY on an explicit user request... **NEVER self-activates**: a bare ulw/ultrawork run, an agent-side routing decision, or reading this file is not a request..."*

### 학습 순서 추천

```
1주차: hooks/*.json 21개       → 생명주기 감각
2주차: components/ulw-loop/,
       lazycodex-executor-verify/ → 검증 메커니즘
3주차: ultrawork/agents/*.toml 12개 → 역할 프롬프트 설계
4주차: skills/*/SKILL.md       → 라우팅 + description 기법
5주차: model-catalog.json      → 비용 설계
```

### 한계점

| 한계 | 설명 |
|---|---|
| `src/` 비어있음 | 서브모듈 초기화 필요 |
| `dist/` 번들 배포 | 컴포넌트가 컴파일된 JS → TS 원본은 OmO 리포에 |
| Codex 종속 | 훅 시스템이 Codex 전용 |
| PR 불가 | 미러 리포라서 PR 자동 차단 |
| beta 버전 | v5.0.0-beta.72 — 잦은 변경 가능 |

---

## 9. React / PHP로 만들 수 있는가

### Part A. 홈페이지 복제 → React 100% 가능 (이미 React)

`packages/web/`이 Next.js 16 + React 19 + Tailwind v4다. 바로 실행 가능:
```bash
cd packages/web
pnpm install && pnpm dev
```
PHP(Laravel + Blade)로도 충분하지만, `ulw-demo` 섹션(Codex 창 애니메이션, React 컴포넌트 7개)은
Alpine.js나 Vanilla JS로 재작성 필요.

### Part B. 하네스 코어 → React 불가, Node.js 필요

| | 가능? | 이유 |
|---|---|---|
| React | 불가 | React는 UI 라이브러리. 훅은 CLI 프로세스로 실행됨 |
| Node.js/TS | 가능 | 원본이 이미 이것 |
| PHP | 이론상 가능 | 훅은 `command` 실행 구조라 PHP CLI도 가능. 다만 MCP SDK 생태계가 TS/Python 중심이고 사용자가 PHP 런타임을 깔아야 함 |

### Part C. 비슷한 제품 신규 제작 → 완전 가능 (권장 전략)

```
[1] 코어 (Node.js/TS) — 필수, 최소한만
    ├── 훅 러너: 생명주기 이벤트 → 스크립트 실행
    ├── 증거 검증기: 산출물 존재/비어있음 체크
    ├── 스킬 로더: SKILL.md 파싱 + description 매칭
    └── 모델 라우터: 난이도 → 모델 매핑

[2] MCP 서버 (TS) — 선택
    └── @modelcontextprotocol/sdk

[3] 대시보드 (React) — ★ 차별화 포인트
    ├── 에이전트 실행 상태 실시간 뷰
    ├── 증거 산출물 갤러리 (스크린샷/diff)
    ├── 토큰 사용량 + 비용 추적 차트
    └── 반복 루프 시각화

[4] 백엔드 (PHP/Laravel) — 팀/SaaS용
    ├── 멤버별 토큰 사용량 집계
    ├── 실행 히스토리 DB
    └── 과금/구독 관리
```

| 보유 스택 | 담당 영역 | 이유 |
|---|---|---|
| **React** | 대시보드 / 시각화 | LazyCodex에 **없는 부분** (GUI 전무) = 차별화 핵심 |
| **PHP** | 팀 관리 / 과금 / API | SaaS 백엔드로 적합 |
| **Node.js** | 훅 러너 / MCP | 불가피하게 필요 (최소한만) |

> **핵심 인사이트: LazyCodex는 CLI만 있고 GUI가 전혀 없다.**
> React로 "에이전트 실행 시각화 대시보드"를 만들면 그것이 바로 수익화 포인트.

---

## 10. 유튜브 강의 제작 가능성 → 매우 적합

### 근거 5개

| 근거 | 설명 |
|---|---|
| 라이선스 OK | MIT → 코드 보여주고 설명 합법 (출처 표기만) |
| 타이밍 | "AI가 거짓 완료 선언" 문제가 현재 개발자 공통 고통 |
| 한국어 공백 | 한국어 Codex 하네스 콘텐츠 거의 없음 = 선점 가능 |
| 시연이 극적 | AI가 스스로 500번 반복하는 장면 = 강한 썸네일 소재 |
| 확장성 | 설치 → 설계 → 응용 → 직접 제작, 시리즈 10편+ 가능 |

### 시리즈 커리큘럼 (10편)

| # | 제목 | 길이 | 난이도 |
|---|---|---|---|
| 1 | AI가 "완료했습니다" 거짓말하는 거 막는 방법 | 12분 | 입문 |
| 2 | 명령어 한 줄로 Codex를 AI 개발팀으로 | 15분 | 입문 |
| 3 | 4대 명령어 완전정복 (`$init-deep`~`$ulw-loop`) | 20분 | 입문 |
| 4 | AI가 혼자 500번 반복하는 걸 지켜봤다 | 18분 | 중급 |
| 5 | 훅(Hook) 21개 해부 — 생명주기의 비밀 | 25분 | 중급 |
| 6 | AI 직원 12명 역할 프롬프트 전부 공개 | 22분 | 중급 |
| 7 | API 요금 줄이는 모델 라우팅 설계 | 16분 | 중급 |
| 8 | 검증된 프롬프트 26개 분석하기 | 20분 | 입문 |
| 9 | 나만의 하네스 직접 만들기 (실습) | 35분 | 고급 |
| 10 | React로 에이전트 대시보드 만들기 | 30분 | 고급 |

### 1편 스크립트 구조

```
00:00-00:20  훅: AI가 "완료" → 실행 → 에러 폭발
00:20-01:30  문제 정의: AI 코딩 도구의 구조적 한계
01:30-04:00  LazyCodex 소개 (LazyVim 비유 + 러시아인형 구조)
04:00-08:00  라이브 시연 (설치 → 훅 승인 → $ulw-loop)
             ★ 하이라이트: Stop 훅 발동 순간
08:00-11:00  원리 해설 (훅 JSON + 프롬프트 원문 공개)
11:00-12:00  마무리 + 다음 편 예고
```

### 타겟 시청자

| 타겟 | 비중 | 후킹 포인트 |
|---|---|---|
| AI 코딩 도구 사용 개발자 | 50% | "거짓 완료 막기" |
| AI 에이전트 제작 희망 개발자 | 30% | "설계도 공개" |
| 개발팀 리더 / CTO | 15% | "비용 절감" |
| AI 관심 비개발자 | 5% | "AI 직원 12명" |

### 제작 시 주의사항

| 주의 | 대응 |
|---|---|
| 출처 표기 필수 | MIT — 설명란에 원본 리포 + 저자(Yeongyu Kim / Sisyphus Labs) 표기 |
| API 키 노출 금지 | 화면 녹화 시 터미널에 키 노출 여부 반드시 확인 |
| 토큰 비용 경고 | "토큰 소비 많음"을 솔직히 고지해야 신뢰 확보 |
| 버전 명시 | v5.0.0-beta.72 기준임을 밝히기 (beta는 자주 변경) |
| 과대광고 금지 | "작은 수정엔 오버킬"을 명시 |

---

## 11. 수익화 아이디어 (상세)

### 대전제

```
LazyCodex 자체를 판매 → 불가능 (MIT 라이선스 = 누구나 무료)

판매 가능한 것 3가지
  1. 지식 (Knowledge)  → 강의, 컨설팅
  2. 시간 (Time)       → 대행, 자동화
  3. 경험 (Experience) → 제품, 서비스
```

> 금광에서 돈 번 사람은 금 캔 사람이 아니라 삽 판 사람이었다.
> LazyCodex는 금광이고, 팔아야 할 것은 삽이다.

### 티어 1 — 즉시 시작 가능 (초기비용 0원)

#### 아이디어 1. 유튜브 + 유료강의 콤보

```
STEP 1 (1~2개월): 유튜브 무료 10편
  목표: 구독자 1,000명, 신뢰 구축
  수익: 애드센스 월 10~50만원

STEP 2 (3개월차): 인프런 유료강의
  "AI 에이전트 하네스 설계 마스터클래스"
  88,000원 / 총 8시간
  100명 수강 = 880만원 (수수료 제외 약 600만원)

STEP 3 (6개월차): 멤버십 / 커뮤니티
  월 19,900원 × 100명 = 월 199만원
  제공: 템플릿, 1:1 Q&A, 신규 패턴 업데이트
```

**차별화:** 다른 콘텐츠는 "사용법"만 다룬다. "설계 원리"(훅 생명주기, 증거 검증, 역할 프롬프트)를 가르치는 것이 핵심.

#### 아이디어 2. 프롬프트/스킬 템플릿 판매

| 상품 | 가격 | 내용 |
|---|---|---|
| Starter Pack | 29,000원 | 스킬 템플릿 10개 + 가이드 PDF |
| Pro Pack | 79,000원 | 26개 전체 + 훅 21개 해설 + 에이전트 12개 TOML |
| Enterprise | 299,000원 | 위 전체 + 커스터마이징 1시간 컨설팅 |

판매 채널: Gumroad, 크몽, 자체 사이트

> 주의: 원본을 그대로 복사 판매는 MIT라도 비매너.
> "분석 + 재구성 + 한국어화 + 독자적 개선"이 반드시 들어가야 상품이 된다.

#### 아이디어 3. 블로그/뉴스레터 → 스폰서십

```
주제: "AI 에이전트 설계 주간 레터"
플랫폼: 스티비, Substack, 자체 구축(PHP)

구독자 1,000명 → AI 툴 회사 스폰서 문의
스폰서 단가: 건당 50~200만원
추가: AI 툴 제휴 링크 커미션
```

### 티어 2 — 중기 (3~6개월, 개발 필요)

#### 아이디어 4. 에이전트 실행 대시보드 (React) — 최우선 추천

**근거: LazyCodex는 CLI만 있고 GUI가 전혀 없다. 명확한 빈틈.**

```
"AgentWatch" (가칭)

[React 프론트엔드]
├── 실시간 실행 모니터     (어느 에이전트가 무엇을 하는지)
├── 증거 갤러리            (스크린샷/diff/테스트결과 타임라인)
├── 토큰·비용 추적 차트    (모델별/작업별 분석)
├── 루프 시각화            (500회 중 몇 번째, 실패 원인)
└── 팀 대시보드            (멤버별 사용량, 성공률)

[PHP/Laravel 백엔드]
├── 사용량 집계 API
├── 구독/과금 (토스페이먼츠, Stripe)
├── 팀/권한 관리
└── 실행 히스토리 DB

[Node.js 수집기 — 최소한만]
└── 커스텀 훅에서 이벤트 수신 → 백엔드 전송
```

| 플랜 | 가격 | 대상 |
|---|---|---|
| Free | 0원 | 개인, 월 100회 실행 |
| Pro | 월 19,000원 | 개인 무제한 |
| Team | 월 9,000원/인 | 5인 이상 팀 |
| Enterprise | 협의 | 온프레미스, SSO |

예상: Pro 200명 + Team 10팀(50인) = **월 830만원**

**기술적 실현성:** 훅이 단순 CLI 명령 실행 구조이므로,
커스텀 훅 하나를 추가해 이벤트를 자체 서버로 전송하면 된다. 난이도 낮음.

#### 아이디어 5. "AI 환각 방어" 플랫폼 (B2B)

```
"ProofGate" (가칭)
핵심 가치: "AI가 만든 코드, 증거 없으면 머지 금지"

기능:
├── CI/CD 통합 (GitHub Actions, GitLab CI)
├── AI 생성 코드 자동 탐지
├── 증거 산출물 강제 (테스트/스크린샷/로그)
├── 증거 없는 PR 자동 차단
└── 감사 로그 (규제 대응)

타겟: 금융, 의료, 공공 — AI 코드 감사 필요 산업
가격: 월 500만원~ (엔터프라이즈)
```

규제 산업은 "AI가 만든 코드를 어떻게 검증했는가"의 증빙이
법적으로 요구되기 시작하는 단계 → 시장 형성 초기.

#### 아이디어 6. 다른 플랫폼 포팅

```
LazyCodex (Codex 전용)
  ↓ 같은 설계를 다른 플랫폼에
LazyClaude  → Claude Code용
LazyGemini  → Gemini CLI용
LazyCursor  → Cursor용

수익: 오픈소스 공개 → 인지도 → 강의/컨설팅/스폰서
     또는 Pro 버전 유료화 (오픈코어 모델)
```

주의: Claude Code엔 이미 공식 스킬/훅 시스템이 있어 차별화 포인트 필요.
"증거 검증 + 대시보드" 조합이면 승산 있음.

### 티어 3 — 고단가 (지식 기반, 즉시 가능)

#### 아이디어 7. 기업 AI 코딩 도입 컨설팅 — 최고 ROI

| 항목 | 내용 |
|---|---|
| 초기 투자 | 0원 (지식만) |
| 단가 | 일 100~300만원 / 프로젝트 1,000~5,000만원 |
| 필요 조건 | 설계 완전 이해 + 포트폴리오 1~2개 |

```
패키지 A: 진단 (2일, 300만원)
  - 현재 AI 도구 사용 실태 분석
  - 토큰 낭비 지점 발굴
  - 개선 로드맵 제시

패키지 B: 구축 (2주, 1,500만원)
  - 하네스 커스텀 구축
  - 사내 스킬/훅 작성
  - 모델 라우팅 최적화 (비용 절감 증빙)

패키지 C: 운영 (월 500만원)
  - 지속 튜닝 / 신규 패턴 적용 / 팀 교육
```

**영업 논리:** "AI 코딩 비용 월 5,000만원 → 모델 라우팅으로 40% 절감
= 월 2,000만원 절감. 컨설팅 비용 500만원. ROI 400%."
숫자로 설득 가능한 것이 강점.

#### 아이디어 8. 기업 세미나 / 사내 교육

```
단가: 반일 200만원 / 하루 350만원 / 3일 과정 900만원
타겟: 개발팀 10인 이상 기업, 국비교육기관, 대학

하루 과정 커리큘럼:
  09:00  AI 코딩 도구의 구조적 한계
  10:30  훅 기반 생명주기 설계 (실습)
  13:00  역할 분리 서브에이전트 (실습)
  15:00  증거 기반 검증 구축 (실습)
  17:00  모델 라우팅 비용 최적화
```

유튜브 10편이 그대로 영업자료가 되어, 기업이 영상을 보고 먼저 연락하는 구조.

#### 아이디어 9. 커스텀 하네스 개발 외주

```
단가: 2,000~8,000만원 / 기간 1~3개월

산출물:
  - 사내 코드베이스 맞춤 스킬 세트
  - 사내 규칙 자동 주입 훅
  - 레거시 시스템 대응 에이전트
  - 사내 위키 연동 MCP 서버
```

### 전체 비교표

| # | 아이디어 | 초기비용 | 기간 | 월수익 잠재력 | 난이도 | 추천도 |
|---|---|---|---|---|---|---|
| 1 | 유튜브+유료강의 | 0원 | 2주 | 100~800만 | 하 | ★★★★★ |
| 2 | 템플릿 판매 | 0원 | 2주 | 50~300만 | 하 | ★★★ |
| 3 | 뉴스레터 | 0원 | 1개월 | 50~200만 | 하 | ★★★ |
| 4 | **React 대시보드 SaaS** | 500만 | 4개월 | 300~2,000만 | 상 | ★★★★★ |
| 5 | B2B 환각방어 | 2,000만 | 8개월 | 1,000~5,000만 | 최상 | ★★★ |
| 6 | 타 플랫폼 포팅 | 0원 | 3개월 | 간접수익 | 상 | ★★★ |
| 7 | **기업 컨설팅** | 0원 | 즉시 | 500~3,000만 | 중 | ★★★★★ |
| 8 | 기업 세미나 | 0원 | 1개월 | 200~1,000만 | 하 | ★★★★ |
| 9 | 커스텀 개발 외주 | 0원 | 즉시 | 프로젝트성 | 상 | ★★★★ |

### 12개월 로드맵

```
1~2개월: 신뢰 자본 쌓기 (투자 0원)
  ├── 유튜브 10편 제작
  ├── 블로그/뉴스레터 시작
  └── 목표 구독자 1,000명 / 예상수익 월 10~50만원

3~4개월: 지식 상품화
  ├── 인프런 유료강의 출시
  ├── 템플릿 팩 출시
  └── 목표 수강생 100명 / 예상수익 월 100~300만원

5~6개월: 고단가 전환 (터닝포인트)
  ├── 기업 컨설팅 영업 (유튜브가 영업자료)
  ├── 세미나 2~3건
  └── 목표 기업 고객 2곳 / 예상수익 월 500~1,000만원

7~12개월: 제품화
  ├── React 대시보드 SaaS MVP
  ├── PHP 백엔드 (과금/팀관리)
  ├── 베타 → 정식 출시
  └── 목표 Pro 200명 / 예상수익 월 1,000~2,000만원
```

### 리스크 & 대응

| 리스크 | 가능성 | 대응 |
|---|---|---|
| OpenAI가 공식 기능으로 흡수 | 높음 | 플랫폼 중립적 설계. "원리"를 팔면 안전 |
| 원본이 먼저 유료화 | 중간 | 현재 MIT지만 변경 가능. 독자 개선 필수 |
| 경쟁자 등장 | 높음 | 선점 + 한국어 특화 + 커뮤니티 방어 |
| 버전 변경으로 콘텐츠 무효화 | 높음 (beta) | "원리" 중심 콘텐츠는 버전 무관 |
| 토큰 비용 부담 | 확실 | 콘텐츠 제작 예산에 미리 반영 |

### 최종 결론

```
1순위: 유튜브 → 컨설팅 루트
  이유: 투자 0원, 즉시 시작, 고단가 연결
  핵심: "사용법"이 아니라 "설계 원리"를 판매

2순위: React 대시보드 SaaS
  이유: LazyCodex의 명확한 빈틈(GUI 없음)
  핵심: React + PHP 백엔드로 완성 가능

3순위: 템플릿/뉴스레터
  이유: 1순위의 부수 수익, 리스크 제로
```

> **가장 중요한 한 가지**
> LazyCodex를 "쓰는 사람"은 수천 명이지만,
> **"왜 이렇게 설계됐는지 설명할 수 있는 사람"은 극소수다.**
> 그 지식 자체가 상품이다.

---

## 12. 부록 — 리포지터리 사실 확인 목록

이 문서의 모든 주장은 아래 파일을 직접 읽어 확인했다.

| 확인 항목 | 근거 파일 |
|---|---|
| 설치 명령어 변환 로직 | `bin/lazycodex-ai.js` |
| 마켓플레이스 등록 | `.agents/plugins/marketplace.json` |
| 플러그인 매니페스트 | `plugins/omo/.codex-plugin/plugin.json` |
| MCP 서버 4개 | `plugins/omo/.mcp.json` |
| 훅 21개 정의 | `plugins/omo/hooks/*.json` |
| 스킬 26개 | `plugins/omo/skills/*/SKILL.md` |
| 에이전트 12개 | `plugins/omo/components/ultrawork/agents/*.toml` |
| 모델 라우팅 | `plugins/omo/model-catalog.json` |
| 텔레메트리 정책 | `plugins/omo/README.md` |
| 설치/인증 문서 | `packages/web/content/docs/installation.md` |
| 설정 문서 | `packages/web/content/docs/configuration.md` |
| 개요 문서 | `packages/web/content/docs/overview.md` |
| FAQ | `packages/web/content/docs/faq.md` |
| 웹 스택 | `packages/web/package.json` |
| PR 자동 차단 | `.github/workflows/pr-source-guidance.yml` |
| 한글 금지 테스트 | `test/no-korean-text.test.mjs` |
| 서브모듈 등록 | `.gitmodules` |

### 알려진 테스트 상태

`test/no-korean-text.test.mjs`는 `.omo/`와 `plugins/omo/` 밖의 추적 파일에
한글이 있으면 실패한다. 이 문서와 `CLAUDE.md`가 그 대상이다.

```bash
$ node --test test/no-korean-text.test.mjs
# fail 1  → offenders: CLAUDE.md, LAZYCODEX-ANALYSIS-KO.md
```

단 `.github/workflows/npm-ci.yml`의 `paths` 필터는
`bin/**`, `test/**`, `plugins/omo/**`, `package.json`, `package-lock.json`만
감시하므로 **루트 마크다운 추가로는 CI가 트리거되지 않는다.**

원본 업스트림에 기여할 계획이 있다면 이 한글 문서들을 제거하거나
`.omo/` 아래로 옮기는 것을 권장한다.
