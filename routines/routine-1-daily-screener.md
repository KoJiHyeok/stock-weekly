ROUTINE ①: 일별 5종목 스크리너 (월~토 22:00 KST, cron 0 22 * * 1-6)
저장소: KoJiHyeok/stock-weekly · 트리거: schedule

너는 매일 밤 미국 상장주 후보 5종목을 조사·3축 채점해 그날 1등을 후보 풀에 적재하고 저장소에 커밋한다. HTML·디스코드 전송은 하지 않는다. 한국어로 기록.

[작업공간] 저장소가 체크아웃된 디렉터리. 먼저 `git pull --rebase` 로 최신화. 핵심 파일: _candidates.json, _state.json(covered_tickers), _review-log.md(있으면).

[STEP 0 — 주차 관리] 오늘이 속한 주의 월요일(YYYY-MM-DD)=weekKey. _candidates.json 의 week_of 가 weekKey 와 다르면 candidates 를 [] 로 비우고 week_of=weekKey 설정.

[STEP 0.5 — 과거 학습] _review-log.md 가 있으면 읽어 과거 예측의 적중/실패 패턴을 채점에 반영(반복 실패 패턴 부합 시 감점, 검증된 성공 패턴이면 가점).

[STEP 1 — 5종목 발굴·조사 (웹검색, 현재 시점 기준)] 시장 전체에서 "저평가 + 성장성 논쟁 + 횡보(박스권)" 테마 미국주 5개 발굴(뉴스·밸류 스크리너·52주 박스권/신저가 부근·실적 종목). covered_tickers 제외, 그 주 이미 적재된 종목과 가급적 비중복. 각 후보: 현재가·52주 위치, 핵심 밸류에이션(PER/선행PER·EV/EBITDA·FCF수익률·배당 중 가능한 것), 성장/촉매 한 줄, 박스권·지지/저항. 핵심 수치 가볍게 교차확인.

[STEP 2 — 3축 채점] 각 후보 value·growth·range(0~10), composite(0~100, 세 축 균형 가중). 데이터 부족 축은 보수적으로. STEP 0.5 학습 반영.

[STEP 3 — 적재 & 커밋] composite 최고 종목을 그날의 픽으로. _candidates.json 의 candidates[] 에 추가: {date, ticker, company, price, value, growth, range, composite, thesis, screened:[5개 티커]}. 그 다음:
  git add _candidates.json
  git commit -m "screener: {date} {TICKER} ({composite})"
  git pull --rebase origin main
  git push origin HEAD:main
끝에 한 줄: "오늘의 후보: TICKER(composite NN) · 스크리닝: A,B,C,D,E".

중요: 변경사항을 반드시 main 브랜치에 직접 반영하라(새 브랜치/PR로 남기지 말 것 — 위 `git push origin HEAD:main`). push 충돌 시 `git pull --rebase origin main` 후 재시도.
데이터를 지어내지 말 것. 근거 기반 채점, 확보 어려운 항목은 보수적으로.
