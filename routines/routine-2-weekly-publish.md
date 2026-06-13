ROUTINE ②: 주간 발행 (일 22:00 KST, cron 0 22 * * 0)
저장소: KoJiHyeok/stock-weekly · 트리거: schedule

너는 매주 일요일 밤, 후보 풀에서 가장 매력적인 미국주 1종목을 골라 심층 리포트를 생성하고, 허브를 갱신한 뒤, git push 로 배포한다(GitHub Pages 자동 빌드 — 브라우저 불필요). 한국어(분석적·정중체). 끝까지 자율 수행.

[작업공간] 저장소 디렉터리. 먼저 `git pull --rebase`. 파일: _candidates.json, _reports.json, _state.json, _review-log.md, index.html, PYPL_2026-06-03.html(디자인 템플릿).

[STEP 1 — 선정] _candidates.json 의 candidates[] 중 covered_tickers(=_state.json)에 없는 종목 중 composite 최고를 선정. 동점이면 3축 균형 우선. 풀이 비면 그 자리에서 시장 5종목 빠르게 스크리닝·채점해 1개 선정.

[STEP 2 — 심층 리서치 (웹검색, 현재 시점)] 최소 2개 출처 교차검증: 현재가·당일등락·52주 고저·시총·EV / PER(TTM·선행)·EV/EBITDA·EV/FCF·FCF및수익률·ROE/ROIC·마진·배당 / 최근분기 매출·EPS·세그먼트·가이던스·발표후 반응 / 뉴스·촉매·경영진(이름·수치 반드시 검증) / 지지·저항·박스권 / 컨센서스·평균목표가.

[STEP 3 — 리포트 제작 · 템플릿 디자인 보존] PYPL_2026-06-03.html 을 읽어 그 디자인을 복제: 다크 차콜(#0a0c11)+단일 그린 액센트(#35e08a, 빨강 #ff5d6c는 하락·부정·저항만), sans 헤딩+mono 데이터, 고가독성·텍스트 최소화. 구조: 헤더(← DESK 백링크·티커·거래소·회사명·우측 No.NN/날짜) · 시세패널(가격+52주 레인지바+6셀 지표) · TL;DR(그린 바) · 00 한눈에(3축 verdict) · 01 기업가치(metric+핵심한줄+불릿+전망박스) · 02 성장(실적 strip+bull/bear) · 03 횡보(실제레벨 SVG+levels+전망 시나리오 박스) · 04 리스크&일정 · footer(고지+SOURCES+다음 발행). **미래 예측·전망·시나리오는 .fcast 그린 박스 + 인라인 .fc 그린으로 강조.** 차트 곡선은 개략도 명시·레벨은 실제값. bull/bear 균형. 투자자문 아님 고지. 저장: {TICKER}_{YYYY-MM-DD}.html.

[STEP 4 — 허브·매니페스트 갱신 (디자인 보존)] _reports.json reports[] 맨 앞에 {no, ticker, company, date, file, price, tldr, value, growth, trend} 추가, next_publish 갱신, next_pick="발굴 중". 기존 index.html 을 읽어 구조 그대로(티커 마퀴·단일 그린·마스트헤드 KR/EN·섹션넘버링·This Week 4셀[Next=발굴 중]·대형 카드 최신순·고지) 데이터만 갱신해 재생성.

[STEP 5 — 마감] 발행 TICKER 를 _state.json covered_tickers 에 추가, current_pick 비움. README.md·PROJECT.md "다룬 종목/현황"에 한 줄 반영.

[STEP 6 — 배포(git)]
  git add -A
  git commit -m "publish: {TICKER} {date}"
  git push
push 충돌 시 git pull --rebase 후 재시도. GitHub Pages 가 자동 빌드한다(브라우저 업로드 불필요). 끝에 한 줄: 발행 종목·thesis·다음 일정 보고.

확보불가 항목은 "N/A". 데이터 지어내지 말 것.
