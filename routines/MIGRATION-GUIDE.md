# Claude Code Routines 마이그레이션 가이드
## WEEKLY EQUITY DESK 자동화를 Anthropic 클라우드로 옮기기

> 목표: 지금 로컬(데스크탑+Chrome 켜짐 필요) 스케줄 3개를 **Claude Code Routines**로 옮겨 Anthropic 서버에서 무인 실행. PC가 꺼져 있어도 매일/매주 자동 작동.

핵심 개선: 클라우드 샌드박스는 **인터넷이 열려 있고 repo를 직접 다룬다** → ① GitHub 배포를 브라우저 업로드가 아닌 **git push**로(Pages 자동 빌드) ② 디스코드를 **웹훅에 직접 curl POST**로(브라우저·discord.com 우회 불필요).

---

## 0. 사전 준비 — 상태 파일을 저장소에 올리기 (1회)

라우틴은 repo를 작업공간으로 쓰므로, 지금 로컬에만 있는 **상태/데이터 파일**도 저장소에 있어야 한다. 저장소 `KoJiHyeok/stock-weekly`에 아래 파일들을 커밋한다(공개돼도 무방한 데이터; **비밀값은 절대 커밋 금지**):

- `_reports.json`, `index.html`, `{TICKER}_*.html` — (이미 있음)
- `_candidates.json`, `_state.json`, `_review-log.md` — **추가 커밋 필요**
- `PYPL_2026-06-03.html`(디자인 템플릿), `PROJECT.md`, `README.md` — 추가 권장

> ⚠️ 디스코드 웹훅 URL은 파일/저장소에 넣지 말 것 → 아래 3번에서 **라우틴 시크릿**으로 등록.

(원하면 이 커밋은 지금 Cowork에서 제가 한 번에 올려드릴 수 있음.)

---

## 1. Claude Code 준비

1. Claude Code 설치 후 로그인(라우틴은 클라우드 기능이므로 지원 플랜 필요 — 본인 플랜의 **하루 라우틴 실행 한도(5~25)** 확인. 우리 사용량은 하루 1~2회라 충분).
2. GitHub 저장소 `KoJiHyeok/stock-weekly`를 Claude Code에 연결(라우틴이 이 repo를 체크아웃해 읽고/커밋/푸시).

---

## 2. 라우틴 3개 등록

각 라우틴 = **프롬프트 + 저장소(stock-weekly) + 스케줄**. 프롬프트는 같은 폴더의 파일을 그대로 붙여넣는다.

| 라우틴 | 프롬프트 파일 | 스케줄 (KST / Asia·Seoul) | cron |
|---|---|---|---|
| ① 일별 스크리너 | `routine-1-daily-screener.md` | 월~토 22:00 | `0 22 * * 1-6` |
| ② 주간 발행 | `routine-2-weekly-publish.md` | 일 22:00 | `0 22 * * 0` |
| ③ 디스코드 보고 | `routine-3-discord-report.md` | 일 23:00 | `0 23 * * 0` |

> 스케줄 타임존을 반드시 **Asia/Seoul(KST)** 로 설정. (라우틴 트리거는 schedule / API / GitHub webhook 중 schedule 사용.)

---

## 3. 시크릿 등록 (디스코드 웹훅)

라우틴 ③이 쓸 디스코드 웹훅을 **라우틴 환경변수/시크릿**으로 등록한다(코드·repo에 노출 안 됨):

- 이름: `DISCORD_WEBHOOK_URL`
- 값: (현재 Cowork의 `equily-agent-discord-report` 작업에 저장된 그 웹훅 URL)
  - 형식: `https://discord.com/api/webhooks/1511746133431685121/****`

라우틴 ③ 프롬프트는 `$DISCORD_WEBHOOK_URL` 를 참조해 curl로 전송한다.

---

## 4. 동작 방식 (옮긴 뒤)

- **월~토 22:00** — ① 스크리너: repo pull → 5종목 채점 → `_candidates.json` 갱신 → commit/push.
- **일 22:00** — ② 발행: 후보 풀 최고점 선정 → 심층 리서치 → 리포트 HTML + 허브 + 매니페스트 갱신 → commit/push (→ GitHub Pages 자동 배포). 브라우저 불필요.
- **일 23:00** — ③ 디스코드: 발행가 대비 변화율 + 지난주 종목 원인해석 → `$DISCORD_WEBHOOK_URL` 로 직접 POST → `_review-log.md` 갱신 commit/push. 브라우저 불필요.

모두 **PC 오프라인이어도 실행**. 데이터·결과는 전부 repo에 남아 추적 가능.

---

## 5. 전환 후 정리

- 라우틴이 정상 작동을 확인하면, 기존 Cowork 스케줄 3개(`weekly-us-stock-daily-research`, `weekly-us-stock-report-publish`, `equily-agent-discord-report`)는 **중복 실행 방지를 위해 비활성화**.
- 디자인·선정 규칙은 `PROJECT.md`와 `PYPL_2026-06-03.html` 템플릿을 단일 기준으로 계속 유지.

---

## 참고 — 로컬(Cowork) vs 클라우드(Routines)

| 항목 | 현재(Cowork 스케줄) | Routines(클라우드) |
|---|---|---|
| 실행 위치 | 내 PC(앱 켜짐 필요) | Anthropic 서버 |
| PC 꺼져 있어도? | ✗ | ✓ |
| GitHub 배포 | 브라우저 업로드 | git push(자동) |
| 디스코드 전송 | 브라우저 fetch | curl 직접 |
| 트리거 | 시간 | 시간 / API / GitHub 웹훅 |
| 한도 | 사실상 무제한 | 하루 5~25회(플랜별) |
