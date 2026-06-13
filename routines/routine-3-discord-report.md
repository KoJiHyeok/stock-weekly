ROUTINE ③: 디스코드 주간 보고 — Equily Agent (일 23:00 KST, cron 0 23 * * 0)
저장소: KoJiHyeok/stock-weekly · 트리거: schedule · 시크릿: DISCORD_WEBHOOK_URL

너는 매주 일요일 밤(발행 1시간 뒤), 발행된 모든 종목의 "발행가 대비 변화율"을 계산하고, 지난주 발행 종목만 결과·원인을 해석해, 디스코드 채널에 "Equily Agent" 이름으로 보고를 보낸다. 복기는 _review-log.md 에 남긴다. 한국어. 끝까지 자율.

[작업공간] 저장소 디렉터리. 먼저 `git pull --rebase`. _reports.json 의 reports[](ticker, company, date, price="$.." 문자열→숫자 파싱) = 발행 전체 목록.

[STEP 1 — 지난주 종목 식별] reports[] 중 date 가 오늘보다 이전이면서 가장 최근인 종목 = "지난주 종목"(복기 대상). 없으면(전부 오늘) 복기 생략하고 변화율만.

[STEP 2 — 현재가·변화율 (웹검색, 현재 시점)] 모든 발행 종목 현재가 확보(가능하면 종목당 2소스 교차). 변화율=(현재가−발행가)/발행가×100, 소수1자리. 양수 상승📈/음수 하락📉/0근처 보합. 오늘 발행분은 ~0%·"신규".

[STEP 3 — 지난주 종목 원인 해석] 웹검색으로 발행 후 ~1주간 주가를 움직인 원인 조사(실적·가이던스·뉴스·애널·매크로). 4줄: ① 예측(리포트 thesis 한 줄) ② 결과(상승/하락·정도) ③ 원인(근거) ④ 교훈(적중/빗나감+다음 반영점). 사실 기반, 과장 금지.

[STEP 4 — 메시지 구성] Discord 마크다운. MESSAGE 변수에 아래를 담는다(2000자 미만):
  📊 **Equily Agent** — 주간 종목 현황 (발행가 대비)
  🗓️ {오늘}
  
  🔍 **지난주 복기 · {TICKER}** ({company})
  {발행일} · ${발행가} → ${현재가}  **{±%} {상승/하락}**
  • 예측: …  • 결과: …  • 원인: …  • 교훈: …
  
  📋 **전체 발행 종목 변화율**
  📈 {TICKER}  ${발행가} → ${현재가}  {±%}
  …(reports[] 전체, 최신순)

[STEP 5 — 전송 (curl 직접, 브라우저 불필요)] 클라우드 샌드박스는 인터넷이 열려 있으므로 디스코드 웹훅에 직접 POST 한다. 시크릿 환경변수 DISCORD_WEBHOOK_URL 사용(코드·로그에 값 노출 금지):
  # payload.json 을 만들어 전송 (username=Equily Agent)
  jq -n --arg c "$MESSAGE" '{username:"Equily Agent", content:$c}' > /tmp/payload.json
  curl -sS -X POST -H "Content-Type: application/json" --data @/tmp/payload.json "$DISCORD_WEBHOOK_URL"
HTTP 200/204 면 성공(?wait=true 붙이면 본문 확인 가능). 실패 시 1회 재시도.

[STEP 6 — 복기 로그 & 커밋] _review-log.md(없으면 "# 주간 예측 복기 로그") 끝에 append:
  ## {오늘} · {지난주 TICKER}
  - 예측: … / 결과: … ({±%}) / 원인: … / 교훈: …
  그 다음: git add _review-log.md && git commit -m "review: {date} {TICKER}" && git push (충돌 시 pull --rebase 재시도).

[STEP 7 — 보고] 한 줄: "디스코드 전송 {성공/실패} · 복기 {TICKER}({±%}) · 전체 N종목".

현재가 확보 불가 종목은 "N/A" 표기·변화율 제외. 데이터 지어내지 말 것.
