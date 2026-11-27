# feed 형식 명세

이 사이트의 리포트·금융권 시각·감시 대상·시장·가격을 정적 JSON 파일로 내보낸다. JavaScript 실행이나 로그인 없이 읽을 수 있다.
사이트를 빌드할 때 화면과 같은 원천에서 함께 만들며, 손으로 따로 관리하지 않는다. 자료는 이 사이트에서 밖으로만 나간다.

- 기준 주소: `<사이트 주소>/feed/`
- 형식 번호: `schema` = 1 (모든 JSON 파일의 맨 위에 있다)

## 파일

| 경로 | 내용 |
| --- | --- |
| `index.json` | 목록. 가장 먼저 읽는 파일 |
| `reports/<슬러그>.json` | 리포트 1건(실적 발표·투자자의 날·인터뷰·글 분석·행사 정리). 비밀번호로 잠근 유형(종합·조사 리포트, `config/site.toml`의 `[sealed]`)은 싣지 않는다 |
| `street.json` | 금융권 시각 최근 30일 항목과 지수 |
| `street/YYYY-MM.json` | 금융권 시각 월별 보관(한국시간 공개 월) |
| `calendar.json` | 실적·투자자의 날 일정과 대기 목록 |
| `health.json` | 수집 상태와 놓친 것(`missed`) |
| `watch.json` | 첫 화면의 감시 대상(AI 설비투자 사이클을 보는 네 갈래의 상태와 그림의 값) |
| `market.json` | 시장 탭(추세 신호·거시 지표·우선주 괴리율·종목 표) |
| `prices.json` | 가격 탭의 GPU 임대가격(채널 기준가·제공자별 값·실거래 지수·계약 기간별 가격) |
| `data/<id>/` | 조사 자료 묶음. 한 질문을 두고 손으로 모은 기록(화면 없음) |
| `SPEC.md` | 이 문서 |
| `QUESTIONS.md` | 판단하기 어려웠던 것 |

증권 리포트 목록과 텔레그램 원문은 내보내지 않는다.

## 파일 규칙

- UTF-8(BOM 없음), 줄바꿈 LF. 한 파일 1.5MB 이하.
- 시각은 한국시간이다(`2026-11-12T09:12:00+09:00`). 날짜만 있는 필드는 `YYYY-MM-DD`.
- 값이 없는 선택 필드는 키를 넣지 않는다. `null` 을 쓰는 필드는 아래에 따로 적었다.
- 리포트를 고치면 `revision` 이 1 오르고 `revised_kst` 와 `review.history` 에 남는다. 조용히 바꾸지 않는다.
- 필드 이름은 바꾸지 않는다. 바꿔야 하면 `schema` 번호를 올리고 이 문서의 변경 이력에 적는다. 값의 종류(예: 출처 `kind`)는 늘어날 수 있다.
- 유료 출처(`access: "paid"`)는 화면과 똑같이 본문을 싣지 않는다. 짧은 인용과 이 사이트의 문장으로 쓴 사실·해석만 나간다. 화면에 없는 것을 feed 에 더 넣지 않는다.
- 이미지·영상은 파일로 넣지 않는다. 원본 주소와 시각(초, `t`)만 넣는다.
- 게시 전에 화면의 리포트 수·금융권 항목 수가 feed 의 수와 같은지, 첫 화면 감시 대상의 항목·상태가 `watch.json` 과 같은지, 시장 탭의 지수·신호·지표·종목이 `market.json` 과 같은지, 가격 탭의 채널 기준가·제공자 칸·실거래 지수·계약 기간별 칸이 `prices.json` 과 같은지 검사한다. 다르면 게시하지 않는다.

## index.json

```json
{
 "schema": 1,
 "generated_kst": "2026-11-12T09:16:31+09:00",
 "reports": [
  {"slug": "coreweave-3q26-earnings", "type": "earnings",
   "title": "…", "companies": [{"name": "CoreWeave", "ticker": "CRWV"}],
   "event": "3Q26 실적 발표", "event_date": "2026-11-10",
   "published_kst": "…", "revised_kst": "…", "revision": 3,
   "summary": "목록에 보이는 요약(해석)", "path": "reports/coreweave-3q26-earnings.json"}
 ],
 "street": {"updated_kst": "…", "path": "street.json"},
 "calendar": {"updated_kst": "…", "path": "calendar.json"},
 "watch": {"updated_kst": "…", "path": "watch.json"},
 "market": {"updated_kst": "…", "path": "market.json"},
 "prices": {"updated_kst": "…", "path": "prices.json"}
}
```

- `generated_kst` 는 빌드 시각이 아니라 **가장 최근에 바뀐 기록의 시각**이다. 내용이 그대로면 이 값도 그대로다.
- `datasets` 는 조사 자료 묶음의 목록이다: `[{"id": "apac-siting", "path": "data/apac-siting/index.json", "updated_kst": "…"}]`. 묶음이 하나도 없으면 키가 없다. `updated_kst` 는 기록이 아직 없으면 `null`.
- `watch.updated_kst` 는 감시 대상이 읽는 기록(실적에서 옮긴 분기 값·거시 지표·가격·종목 표) 가운데 가장 최근에 바뀐 시각이다.
- `market.updated_kst` 는 시장 탭의 기록(추세 신호·거시 지표·우선주 괴리율·종목 표) 가운데 가장 최근에 바뀐 시각이다. 장중·장 마감 뒤 실행마다 바뀐다.
- `type`: `earnings` | `investor_day` | `interview` | `article` | `event`(행사 정리). 종합(`synthesis`)·조사 리포트(`research`)는 잠가 싣지 않는다.
- `companies[].ticker` 는 없으면 `null`. `event`·`event_date` 는 있을 때만.
- `revision` 은 처음 게시할 때 1이다. `revised_kst` 는 고친 적이 없으면 `published_kst` 와 같다.
- `path` 는 `feed/` 기준 상대 경로다.

## reports/<슬러그>.json

```json
{
 "schema": 1, "slug": "…", "type": "earnings", "title": "…", "summary": "…",
 "companies": [{"name": "CoreWeave", "ticker": "CRWV"}], "event": "…", "event_date": "…",
 "period": "3Q26", "period_end": "2026-09-30",
 "prev_slug": null,
 "published_kst": "…", "revised_kst": "…", "revision": 3,
 "sources": [
  {"n": 1, "title": "CoreWeave 3Q26 실적 보도자료", "url": "…", "date": "2026-11-10",
   "access": "public", "kind": "press_release", "read": "전문", "note": "…"}
 ],
 "sections": [
  {"id": "conclusion", "h": "한 문장 결론", "layer": "common",
   "blocks": [
    {"kind": "view", "text": "…"},
    {"kind": "quote", "speaker": "…", "role": "CFO", "text": "한국어 옮김", "orig": "원어 원문", "src": 2, "loc": "준비 발언 — CFO"},
    {"kind": "fact", "items": ["…", "…"], "src": 1, "loc": "…"},
    {"kind": "fact_table", "title": "…", "head": ["항목", "3Q26", "3Q25"], "rows": [["매출", "3,661", "1,365"]], "note": "…", "src": 1},
    {"kind": "calc", "text": "결과 문장", "inputs": [{"name": "…", "value": "…"}], "formula": "…"},
    {"kind": "assumption", "by": "ai", "text": "…"},
    {"kind": "explain", "text": "…"},
    {"kind": "qa", "topic": "…", "turns": [
      {"q": true, "speaker": "…", "role": "애널리스트", "text": "…"},
      {"q": false, "speaker": "…", "role": "CFO", "text": "…"}], "src": 2, "loc": "Q&A 6"}
   ]}
 ],
 "capacity": [{"label": "가동 전력", "value": 1.7, "unit": "GW", "basis": "unknown", "asof": "2026-09-30", "src": 1}],
 "watch_next": ["다음 분기에 확인할 지표 문장", "…"],
 "review": {"draft": {"model": "…", "at_kst": "…"}, "review": {"model": "…", "at_kst": "…", "result": "통과"},
            "summary": "…", "history": [{"at_kst": "…", "text": "…"}]}
}
```

### 두 겹: `layer`

- `common` — 공통 본문. 누구의 관점도 넣지 않은 중립 본문이다. 사실·발언·계산과, 원문에서 바로 따라 나오는 해석까지.
- `investor` — 투자자 관점. 밸류에이션·목표주가·컨센서스 추정 방향·종합 투자 판단. 공통 본문 뒤의 덧붙임이며 파일에는 표시만 해 두고 넣는다. 받아 쓰는 쪽이 골라 쓴다.

### 절

실적 발표와 투자자의 날은 절의 `id`·이름·순서가 고정이다. 마지막 `investor_view` 만 없을 수 있다.

| `id` | 실적 발표(`earnings`) | 투자자의 날(`investor_day`) | `layer` |
| --- | --- | --- | --- |
| `conclusion` | 한 문장 결론 | 한 문장 결론 | common |
| `numbers` | 숫자 요약 | 중기 목표 요약 | common |
| `primer` | 한눈에 이해하기 | 한눈에 이해하기 | common |
| `prepared` | 콜 정리 — 준비 발언 | 행사 정리 — 발표 | common |
| `qa` | 콜 정리 — 질의응답 | 행사 정리 — 질의응답 | common |
| `positives` | 긍정적이었던 내용 | 새로 나온 것 | common |
| `negatives` | 부정적이었던 내용 | 빠지거나 물러난 것 | common |
| `caveats` | 읽을 때 걸러 들을 점 | 읽을 때 걸러 들을 점 | common |
| `watch_next` | 다음 분기에 확인할 지표 | 앞으로 분기 실적에서 확인할 지표 | common |
| `investor_view` | 투자자 관점 | 투자자 관점 | investor |

- 전사본이 없어 콜 정리를 싣지 못한 리포트도 `prepared`·`qa` 절은 있다. 그 사실을 `view` 블록 하나로 적는다.
- 인터뷰·글 분석·행사 정리는 화면의 절을 그대로 내보낸다. `id` 는 순번(`s1`, `s2` …)이고 `layer` 는 모두 `common` 이다.
- 종합·조사 리포트(`synthesis`·`research`)는 비밀번호로 잠가 `feed/` 에 싣지 않는다(`config/site.toml` 의 `[sealed]`). 조사 자료 묶음(`data/<id>/`)은 그대로 나가고, 묶음에서 쓴 조사 리포트는 자체 분석 탭에서 잠겨 있다.
- 출처 목록(`sources`)과 작성·검토 기록(`review`)은 절이 아니라 최상위 필드다.

### 블록 `kind`

| `kind` | 내용 | 필드 |
| --- | --- | --- |
| `quote` | 화자의 발언·원문 인용 | `text`(한국어), `speaker`, `role`, `orig`(원어 원문. 정확한 문구가 중요한 발언에만), `src`, `loc`, `t` |
| `qa` | 질문과 답변 한 묶음 | `topic`, `turns[]`(`q`: 질문이면 true, `speaker`, `role`, `text`), `src`, `loc`, `t` |
| `fact` | 원문에서 확인한 수치·사실 | `text` 또는 `items`(둘 다일 수 있다), `src`, `loc`, `t` |
| `fact_table` | 원문에서 확인한 값의 표 | `title`, `head`, `rows`, `note`, `src`, `loc`, `t` |
| `calc_table` | 계산한 값의 표 | `title`, `head`, `rows`, `note` |
| `view_table` | 해석을 정리한 표 | `title`, `head`, `rows`, `note` |
| `fact_chart` · `calc_chart` · `view_chart` | 그림의 값(막대 `bars`, 하한·중앙·상한 `range`, 꺾은선 `line`) | `form`, `title`, `unit`, `labels`, `series`(`[{label, values}]`), `note`, `src`, `loc` |
| `calc` | 가정과 사실로 계산한 값 | `text`, `inputs[]`(`name`, `value`), `formula` |
| `assumption` | 계산·판단의 전제 | `by`(`ai` \| `user`), `text` |
| `explain` | 배경 설명(AI) | `text` 또는 `items` |
| `view` | 해석·의견(AI) | `text` 또는 `items` |

- `src` 는 `sources[].n` 이다. `loc` 은 원문 안 위치(쪽, 소제목, `Q&A 3`, 영상 시각 `[12:34]`)다.
- `t` 는 `loc` 에 영상 시각이 있을 때 그 시작 시각(초)이다.
- `text` 는 빈 줄로 문단을 나누고 강조는 `**굵게**` 만 쓴다.

### 그 밖의 필드

- `sources[].access`: `public` | `paid` | `private`(공개 링크가 없는 출처. `url` 이 없다).
- `sources[].kind`: `press_release` | `transcript_official` | `transcript_third_party` | `presentation` | `market_data` | `prior_report` | `article` | `video` | `news`(언론 보도) | `telegram`(텔레그램 글) | `desk`(데스크 기록) | `dataset`(조사 자료 묶음 파일. `url` 은 `data/<id>/` 의 파일이고 값마다의 원 출처는 그 파일의 기록에 있다). 종류가 적혀 있지 않은 인터뷰·글 분석의 출처는 주소가 유튜브면 `video`, 아니면 `article` 로 낸다.
- `sources[].read`: 실제로 읽은 범위. `전문` | `일부` | `요약만`.
- `period` 는 회사가 쓰는 분기 이름이다(회계연도가 달력과 다른 회사 포함). `period_end` 는 그 분기의 말일이라 회사끼리 견줄 때 쓴다. 둘 다 있을 때만 들어간다.
- `prev_slug`: 이 사이트에 직전 분기 리포트가 있으면 그 슬러그, 없으면 `null`.
- `capacity`: 리포트에 나온 데이터센터·전력 용량 값. 있을 때만. `basis`: `it`(IT 용량) | `utility`(IT 용량 없이 수전 용량만 공개됨) | `unknown`(회사가 기준을 밝히지 않음). 수전 값을 IT 로 환산한 값은 넣지 않는다.
- `watch_next`: `watch_next` 절의 항목을 문자열 배열로 다시 낸 것이다. 그 절이 있는 리포트에만 있다.
- `review.summary` 는 검토 요약이 없으면 `null`.

## street.json

```json
{
 "schema": 1, "updated_kst": "…", "window_days": 14, "items_days": 30,
 "index": {"ai_tech": {"value": -1, "label": "엇갈림", "orgs": 30, "items": 76},
           "market":  {"value": -12, "label": "엇갈림", "orgs": 23, "items": 46},
           "by_group": [{"group": "벤처캐피털", "ai_tech": 42, "ai_tech_orgs": 5, "market": null, "market_orgs": 0}],
           "weekly": [{"week": "2026-11-09", "ai_tech": 5, "ai_tech_orgs": 18, "market": -14, "market_orgs": 16}],
           "args": {"caution": [{"tag": "투자 회수", "n": 15}], "optimism": [{"tag": "수요·채택", "n": 23}]},
           "method": "기관·인물마다 기간 안 입장 값의 평균을 내고 그 평균들을 다시 평균해 ×50"},
 "items": [
  {"id": "…", "group": "투자 거장", "org": "Oaktree Capital", "person": "Howard Marks",
   "kind": "report", "title": "한국어 제목", "title_orig": "원문 제목", "url": "…", "outlet": "…", "published_kst": "…",
   "stance": {"ai_tech": -1, "market": -1}, "args": ["쏠림", "자금 조달·신용"],
   "summary": "…", "points": ["…", "…"], "evidence": [{"ko": "…", "orig": "…"}],
   "judged": {"basis": "full_text", "tier": "standard"}}
 ]
}
```

- `index` 는 최근 `window_days` 일의 항목으로 계산한 값이고, `items` 는 최근 `items_days` 일의 항목이다.
- 지수 값은 −100(강한 경계) ~ +100(강한 낙관). 기관·인물 수가 모자라 값을 내지 않으면 `null`.
- `index.weekly[].week` 는 한국시간 월요일 날짜다.
- `kind`: `publication`(기관·인물이 직접 낸 글·영상) | `report`(언론이 전한 발언).
- `stance` 값은 −2 ~ +2 정수, 다루지 않았으면 `null`. 입장 값·논거·요약은 모두 AI 의 해석이다.
- `person` 은 없으면 `null`. `title_orig` 은 한국어 번역 제목이 있을 때만 있다.
- `summary`·`points` 는 본문을 읽은 항목에만 있다. `evidence` 의 원문 문장은 화면에 함께 보여 주는 것만 나간다.
- `judged.basis`: `full_text`(전문) | `partial`(길어서 앞부분만) | `headline_only`(제목과 검색 요약문만).
- `street/YYYY-MM.json` 은 `{"schema", "month", "updated_kst", "items"}` 이고 `items` 의 모양은 위와 같다. 모든 항목이 월별 보관 어딘가에 한 번씩 들어 있다.

## calendar.json

```json
{
 "schema": 1, "updated_kst": "…",
 "events": [{"company": "Nvidia", "ticker": "NVDA", "type": "earnings", "period": null,
             "date": "2026-11-18", "confirmed": true, "official": true, "source_url": "…",
             "consensus_snapshot": true, "report_slug": null, "status": "scheduled"}]
}
```

- `type`: `earnings` | `investor_day`.
- `date` 는 **한국시간 기준 날짜**다. 미국 장 마감 뒤 발표는 미국 날짜보다 하루 늦게 적힌다.
- `confirmed`: 실적 발표일은 야후 파이낸스가 추정이 아니라고 표시한 날짜만 `true` 다. 손으로 넣은 일정은 출처를 확인한 것이다.
- `official`: 회사 IR(행사 달력, 일정 안내 보도자료)에서 확인한 날짜면 `true`, 야후 파이낸스 표기면 `false` 다. `true` 일 때 `source_url` 은 그 회사 페이지다.
- `period`: 분기 이름을 아직 모르면 `null`. `ticker` 는 자기 주식이 거래되지 않는 회사면 `null`.
- `consensus_snapshot`: 발표 전 7일 안에 받아 둔 주가·컨센서스 스냅샷이 있는가. 없으면 리포트에 "예상 대비"를 쓰지 않는다.
- `status`
  - `scheduled` — 아직 열리지 않음
  - `queued` — 열렸고 리포트를 기다리는 대기 목록에 있음
  - `waiting_transcript` — 리포트는 냈지만 전사본 없이 보도자료만으로 씀
  - `published` — 전사본까지 읽은 리포트가 있음(`report_slug`)
  - `missed` — 최근 7일 안에 열렸는데 리포트도 없고 대기 목록에도 오르지 않음
- 지난 일정은 120일 동안 남는다.

## health.json

```json
{
 "schema": 1, "updated_kst": "…",
 "collectors": [{"name": "reports", "items": 7, "last_kst": "…"},
                {"name": "street", "items": 103, "last_kst": "…"},
                {"name": "calendar", "items": 33, "last_kst": "…"}],
 "missed": [{"company": "…", "ticker": "…", "type": "earnings", "period": null, "date": "…", "reason": "…"}]
}
```

- `collectors[].last_kst` 는 그 기록이 마지막으로 바뀐 시각이다. 오래 멈춰 있으면 수집이 돌지 않은 것이다.
- `missed` 는 `calendar.json` 에서 `status` 가 `missed` 인 일정이다.

## watch.json

첫 화면 아래의 감시 대상을 그대로 낸다. 사이트 빌드가 화면을 그릴 때와 같은 계산(`site/src/lib/watch.ts`)의 결과이고, 신호의 상태는 따로 저장하지 않으므로 이 파일이 화면 밖에서 상태를 읽는 유일한 곳이다. 기록이 없으면 `updated_kst` 가 `null` 이고 `items` 가 빈 목록이다.

```json
{
 "schema": 1, "updated_kst": "…", "unit": "십억 달러",
 "quarters": [{"end": "2025-12", "label": "4Q25"}, {"end": "2026-09", "label": "3Q26"}],
 "rules": {"turn_count": 2, "high_pos": 90, "decel_pt": 1},
 "labels": {"state": {"ok": "유지", "warn": "경계", "turn": "꺾임", "none": "보류"}, "move": {…}, "stance": {…}, "verdict": {…}},
 "items": [
  {"id": "capex", "no": 1, "title": "빅테크 설비투자 가이던스", "question": "…", "state": "warn", "headline": "…",
   "signals": [{"name": "마이크로소프트", "state": "warn", "text": "…"}],
   "theses": [{"id": "…", "title": "…", "statement": "…", "verdict": "mixed", "recent": [{"when": "2026-11-20", "daily": true, "verdict": "mixed"}]}],
   "gaps": ["…"],
   "companies": [{"id": "microsoft", "name": "마이크로소프트", "basis": "금융리스 포함", "report": "microsoft-1q27-earnings",
                  "capex": [37.5, 31.9, 41, 45.2], "why_blank": ["…", null, null, null],
                  "guidance": {"label": "…", "move": "down", "prior": {"label": "…", "low": 0, "high": 0},
                               "history": [{"date": "2026-10-28", "low": 170, "high": 170, "bound": "약", "note": "…", "report": "…"}]},
                  "next_year": {"label": "2027년", "stance": "up", "text": "…", "date": "…", "report": "…"},
                  "next_earnings": {"date": "2027-01-28", "confirmed": false, "period": null},
                  "signal": {"state": "warn", "text": "…"}}],
   "total": {"quarter": "2026-09", "value": 0}}
 ]
}
```

- 항목은 화면 순서대로 넷이다: `capex`(빅테크 설비투자 가이던스) · `financing`(자금 조달 구조) · `demand`(최종 수요 매출) · `supply`(공급망 선행지표).
- 무엇이 무엇인가: `question` 은 사용자가 적은 물음이다. 값(`capex`·`values`·`growth`·스프레드·가격·가이던스)은 확인된 사실이다. 상태(`state`)·`signals`·`headline`·`total`·`pace`·`fcf_note` 는 규칙으로 계산한 값이다(AI 없음). `theses[].verdict` 는 AI 의 해석이다. 상태와 가설 판정은 섞지 않는다.
- `state`: `ok`(유지) | `warn`(경계) | `turn`(꺾임) | `none`(보류, 판단할 값이 없음). 항목의 상태는 신호 가운데 꺾임이 `rules.turn_count` 개 이상이면 꺾임, 하나라도 경계·꺾임이면 경계, 그 밖에는 유지다. 신호마다의 규칙은 첫 화면의 "상태를 정하는 규칙"과 `docs/areas/digest.md` 의 "감시 대상"에 있고, 문턱값은 `rules`(`high_pos`: 52주 범위 안 자리 %, `decel_pt`: 성장률 둔화 %p)와 행마다의 `widen_bp`·`drop_pct` 다.
- `quarters[].end` 는 분기 말의 달이다. 분기 값의 목록(`capex`·`values`·`growth`)은 `quarters` 와 길이·순서가 같고, 회사가 밝히지 않은 분기는 `null` 이다. 그 이유는 같은 자리의 `why_blank` 에 있다(빈 칸이 없으면 키가 없다). 화면처럼 최근 여섯 분기까지만 나간다. 금액의 단위는 `unit`(십억 달러)이다.
- `report` 는 그 값을 옮겨 온 실적 리포트의 슬러그(`reports/<슬러그>.json`)다. 없으면 `null`.
- `theses[].verdict`: `supported` | `mixed` | `weakened` | `broken` | `null`(아직 점검 전). `recent` 는 자료가 있던 최근 점검 여덟 번까지이고, `when` 은 일일 점검이면 날짜, 주간 점검이면 `2026-W46` 같은 주다.
- `labels` 는 화면에 쓰는 한국어 이름이다(상태·가이던스 변화·내년 계획·가설 판정).
- `capex.companies`
  - `guidance.history` 는 올해 들어 제시한 설비투자 가이던스의 변화이고 마지막이 지금 값이다. `low`·`high` 는 범위의 양 끝(하나의 값이면 같다), `bound` 는 "약"·"이상" 같은 말이다.
  - `guidance.move`: `up` | `same` | `down` | `first`. `prior` 는 회사가 새 연도의 가이던스로 넘어왔을 때 지난 계획의 마지막 값이다.
  - `next_year.stance`: `up`(증가 예고) | `flat`(동결) | `down`(축소) | `none`(미제시).
  - `next_earnings` 는 종목 표의 다음 실적일이고, 없으면 `null` 이다.
  - `total` 은 가장 최근 분기의 합계다. 회사마다 기준(`basis`)이 달라 크기를 가늠하는 값이다. 한 곳이라도 값이 없으면 `null` 이다.
- `financing`
  - `spreads` 는 미국 IG·HY 스프레드다. `value` 는 `unit`(%p), `m1_bp`·`m3_bp` 는 한 달·석 달 변화(bp), `weekly` 는 최근 1년의 주간 값(마지막이 `date`), `pos` 는 52주 범위(`range_52w`) 안의 자리(%)다.
  - `ai_spreads` 는 AI 발행사의 회사채 스프레드로, 발행사마다 채권 하나다. 채권의 독일 거래소 종가로 계산한 만기수익률(`ytm`, %)에서 남은 만기가 같은 미국 국채 수익률을 뺀 값이다. `value` 는 %p 이고 화면은 ×100 해 bp 로 보인다. `url` 은 종가를 받은 곳이다. `reference: true` 인 줄은 표에만 보이는 참고 값이라 `widen_bp` 가 없고 `signal.state` 가 `none` 이며 항목의 `signals`·상태·머리줄의 집계에 들지 않는다. `note` 는 참고로만 두는 까닭이다(일반 회사채가 없어 전환사채로 계산한 발행사).
  - `fcf` 는 분기 잉여현금흐름이다. `debt` 는 부채 잔액이고, 화면처럼 첫 분기(`start`)와 가장 최근 분기(`end`)만 나간다.
- `demand`
  - `clouds[].growth` 는 클라우드 사업의 전년 대비 매출 성장률(%)이다. `label` 은 사업 이름이다(마이크로소프트는 Azure 금액을 밝히지 않아 Microsoft Cloud).
  - `pace` 는 설비투자와 클라우드 매출이 함께 있는 첫 분기(`from`)부터 가장 최근 분기(`to`)까지의 증가 배수(`capex_x`·`revenue_x`)다. `lag` 은 매출이 더 느린가다. 설비투자는 회사 전체이고 계절을 맞추지 않았다.
  - `labs` 는 AI 랩의 연환산 매출(회사 정의 지표)이다. 조사 자료 묶음 `data/ai-knowledge-work/spend.json` 의 기록에서 읽는다. 값을 비운 랩은 `value` 가 `null` 이고, 보도된 범위(`range`, 원문 미확인)가 있으면 함께 나간다.
  - `usage` 는 회사가 실적 발표에서 밝힌 AI 사용량이다(분기 값, 사실). `metric` 은 `api_tokens`(모델 API 처리 토큰, `unit` 분당 십억 개) | `app_users`(AI 앱 사용자, `unit` 억 명), `values` 는 `quarters` 와 같은 순서이고 빈 칸의 이유는 `why_blank` 에 있다. `basis` 에 정의와 회사의 표현("이상"·"약", MAU·사용자)이 적혀 있다. 지금은 알파벳만 있다. `signal` 은 가장 최근 값이 앞 값보다 낮으면 꺾임이다. 실적 발표라 1~2개월 늦게 나오는 확인용이어서 `demand.signals` 와 항목 상태에는 들어가지 않는다(화면에서도 접어 둔다).
  - `ramp` 는 기업의 AI 결제(Ramp AI Index, 미국 기업의 카드·매입 결제)다(설정이 없으면 `null`). `growth.months` 는 최근 달(`YYYY-MM`), `growth.rows[]` 는 회사(`vendor`, 원천의 이름)마다 기업 토큰 결제액의 전월 대비(`values_pct`, %)와 `signal`(가장 최근 달이 마이너스면 경계, 두 달 연속이면 꺾임 — `demand.signals` 에 들어간다). `adoption`·`per_employee` 는 가장 최근 달(`month`)의 값과 전월 대비 차(`rows[]` 의 `name`·`value`·`change`; 비중은 %, 1인당 지출은 달러), `token_index` 는 가장 최근 주(`week`, 주가 끝나는 일요일)의 사용량·결제액 지수(2026-01-11 주 = 100)다. 원천이 판마다 지난 값을 고친다.
  - `capacity` 는 용량 압박 — 회사 상태 페이지의 장애다(설정이 없으면 `null`, 상태에 넣지 않는다). `weeks` 는 끝난 주의 시작일(월요일), `rows[]` 는 회사(`vendor`)마다 `counts`·`hours`(주마다 장애 건수·시간, 기록이 덮지 않는 주는 `null`), `recent`(최근 `recent_weeks` 주의 합 `count`·`hours`, 모자라면 `null`), `prior_avg`(그 앞 주들의 주 평균 건수), `since`(기록 시작), `last`(가장 최근 장애)다. 회사마다 장애를 올리는 기준이 달라 회사끼리 견주지 않는다.
  - `tokens` 는 토큰 수요의 1월 판정이다(설정이 없으면 `null`). 연말 휴가로 빠진 사용량이 1월에 휴가 전보다 크게 늘어나는지 본다. `year` 는 판정하는 1월의 해(12월부터 다음 해로 넘어간다), `base_weeks`(휴가 전 기준선, 전년 12월 초 두 주)·`window_weeks`(판정 창, 1월 둘째·셋째 주)·`first_week`(1월 첫 주, 보이기만 한다)는 주 시작일(월요일)이다. `first_date`·`verdict_date` 는 첫 주 값과 판정이 나오는 날(그 주가 끝난 다음 월요일), `next_verdict` 는 다음 해의 판정일이다. `ref_pct` 는 재가속으로 보는 기준(%)이고 `ref_from` 이 `last_year`(작년 같은 계산) 또는 `default`(설정의 `reaccel_pct`)다. `flat_pct` 아래면 꺾임, `share_drop_pt` 는 클로즈드 모델 안의 몫(`share_pct.base` → `share_pct.window`, %)이 이만큼 넘게 줄면 경계로 보는 폭이다.
    - `rows` 의 첫 줄(`main: true`)이 상태를 정하는 계열(OpenRouter 의 앤트로픽 주간 토큰)이고 나머지는 같은 계산의 참고다. `kind` 는 `openrouter`(OpenRouter 제작사별 주간 토큰) | `openrouter-spend`(OpenRouter 날짜별 기록의 제작사별 추정 결제액 — 정가 기준 상한의 주 합, 달러) | `ramp`(Ramp 의 주간 기업 토큰 사용량·결제액 지수) | `dev`(개발자·사용자 활동의 공개 흔적: 하루 값을 월요일 주로 묶은 것 — 하루 수는 합, 순위·30일 설치는 평균, 누적은 일요일 값끼리의 차), `source` 는 원천 이름(`OpenRouter` | `GitHub`(공개 저장소) | `npm` | `PyPI` | `Homebrew` | `VS Code` | `앱스토어 미국`), `unit` 은 `토큰/주` | `커밋/주` | `PR/주` | `다운로드/주` | `30일 설치` | `설치/주` | `새 평가/주` | `무료 순위` 다. `base`·`first`·`window` 는 주 값(두 주면 평균), `ratio_pct` 는 판정 창 ÷ 기준선 − 1(%), `last_year_pct` 는 작년 같은 계산, `yoy` 는 가장 최근 완결 주(`week`)의 52주 전 같은 주 대비(%)다. `measure` 가 `rank`(앱 순위)인 줄은 값이 주 평균 순위(작을수록 위)이고 `ratio_pct`·`last_year_pct`·`yoy.pct` 가 % 가 아니라 올라간 계단 수(앞 순위 − 뒤 순위)다. 그 밖의 줄은 `volume` 이다. `vendor` 는 접어 둔 표에서 줄을 묶는 회사(`anthropic` | `openai` | `google`)이고, `group` 은 펼쳐 둔 표의 묶음 이름(OpenRouter 토큰·결제액, 기업 결제)이다. 없으면 `null`. 값이 모자라면 `null` 이다. `trend` 는 마지막 주(`trend_end`)부터 거슬러 끊김 없이 이어진 주 값(최대 26주. 빈 주가 있으면 그 뒤만), `rough` 는 계산에 쓴 주에 대략 값(검색이 시간 안에 끝나지 못한 날의 커밋 수)이 들었는가다.
    - 다운로드는 자동 빌드·에이전트 실행 환경이 부풀리고 공개 저장소의 커밋은 연말에 오히려 늘어, 수준이 아니라 같은 계열 안의 달력 변화를 보는 값이다.
- `supply.indicators`
  - `group`: `GPU 임대가` | `메모리`. `m1_pct`·`m3_pct` 는 한 달·석 달 변화(%)이고, 기록이 짧아 아직 없으면 `null` 이다. 그 사정은 `short` 에 적는다.
  - `trend` 는 화면의 작은 꺾은선 값이고, `trend_step` 은 그 간격(`day` | `week` | `month`)이다.
  - GPU 임대가는 모델 × 채널의 기준가(GPU 한 장 · 시간당 달러)다. 수출물가지수는 한국은행(2020=100), 현물가·고정거래가는 TrendForce, 서버 RDIMM 월간 시세는 CFM 이다. 전체 표는 가격 탭에 있다.

## market.json

시장 탭(`market/`)을 낸다. 값은 `content/market` 의 기록 그대로이고 사이트는 계산하지 않는다(계산은 Python, `docs/areas/market.md`). 구획은 화면 순서대로 넷이고, 기록이 없는 구획은 `null` 이다. 탭의 가격 한 줄은 가격 탭의 요약이라 넣지 않고(GPU 임대가·메모리는 `watch.json` 의 `supply.indicators`), 관심 종목 아래의 관련 자료 목록도 넣지 않는다.

```json
{
 "schema": 1, "updated_kst": "…",
 "trend": {"updated_kst": "…", "rule": {"days": 200, "band_pct": 3, "share_pct": 50, "cost_pct": 0.1}, "labels": {"state": {"stock": "주식", "cash": "현금"}},
  "assets": [{"id": "sp500", "label": "S&P 500", "source": {"name": "Yahoo Finance", "series": "^GSPC", "url": "…"},
   "date": "2026-11-25", "close": 7182.78, "from_high_pct": -8.1, "state": "cash", "average": 7352.49, "line": 7573.07, "gap_pct": -5.2,
   "since": {"date": "2026-11-20", "trade_date": "2026-11-23", "trade_close": 7237.17, "change_pct": -0.8},
   "turns": [{"date": "…", "to": "stock", "close": 0, "trade_date": "…", "trade_close": 0, "result_pct": null}],
   "daily": [["2024-11-26", 5987.4, 5612.3, 5444.0]],
   "backtest": {"from": "1950-10-19", "to": "…", "years": 76.1, "in_stock_pct": 72, "trades_per_year": 1.0,
                "hold": {"cagr": 8.0, "mdd": -56.8}, "mixed": {…}, "trend": {…}, "cash": "TB3MS"},
   "sells": {"count": 39, "rebought_higher": 29, "avg_higher": 7.4, "avg_lower": -12.6, "best": -34.3}, "missing": {}}],
  "missing": {}},
 "macro": {"updated_kst": "…", "groups": [{"key": "rates", "label": "금리"}],
  "indicators": [{"id": "us10y", "label": "미국 10년물", "group": "rates", "value": 4.91, "unit": "%", "decimals": 2, "date": "2026-11-24",
   "change": "bp", "delta": {"d1": -1, "w1": -19, "m1": -9, "m3": 21, "ytd": 73}, "range_52w": {"low": 3.97, "high": 5.31},
   "weekly": [4.4, …], "home": true, "source": {"name": "FRED", "series": "DGS10", "url": "…"}}],
  "missing": {}},
 "pref_gap": {"updated_kst": "…", "rule": {"anchor_days": 365, "daily_days": 500}, "source": {"name": "네이버 금융", "url": "…"},
  "pairs": [{"id": "samsung", "label": "삼성전자", "common": {"code": "005930", "close": 0, "url": "…"}, "pref": {…},
   "date": "…", "gap_pct": 21.63, "foreign_pct": {"common": 45.84, "pref": 74.54},
   "periods": [{"label": "전체", "from": "1990-03-02", "days": 9466, "mean": 33.9, "median": 31.6,
                "min": {"date": "…", "value": 4.8}, "max": {"date": "…", "value": 78.2}, "pct": 28}],
   "to_median_pct": 5.8, "anchor": {"date": "…", "gap": 37.46, "common_change": -45.6, "pref_change": -31.9, "beta_pref_change": -40.7},
   "beta": {"value": 0.858, "from": "…", "to": "…", "days": 1975}, "beta_gap_pct": 31.81,
   "monthly": [["1990-03", 40.1]], "daily": [["2024-11-26", 18.2, null]], "missing": {}}],
  "missing": {}},
 "board": {"updated_kst": "…", "as_of": "2026-11-25", "sectors": [{"key": "hyperscaler", "label": "하이퍼스케일러"}],
  "fx": {"base": "USD", "rates": {"EUR": {"rate": 0.91675, "date": "…"}}},
  "companies": [{"id": "microsoft", "name": "마이크로소프트", "ticker": "MSFT", "sector": "hyperscaler", "watch": false, "snapshot_date": "…",
   "price": {"close": 489.93, "date": "…", "currency": "USD"}, "market_cap": 0, "market_cap_usd": 0,
   "returns": {"d1": -1.25, "w1": -3.61, "m1": -6.59, "m3": -0.36, "ytd": 1.3}, "from_high_pct": -11.85, "range_52w": {"low": 0, "high": 0},
   "per": {"ntm": 22.5, "fy0": 24.5, "fy1": 20.2}, "fy": {"fy0_end": "2027-06-30", "fy1_end": "2028-06-30"},
   "eps": {"fy0": 0, "fy1": 0, "ntm": 0, "growth": 20.97, "currency": "USD"}, "revision": {"basis": "ntm", "d30": 1.74, "d90": 2.01},
   "revenue": {"growth": 20.72, "psr_ntm": 8.6}, "target": {"mean": 0, "upside": 28.47},
   "next_earnings": {"date": "2027-01-28", "confirmed": false, "period": null}, "report": "microsoft-1q27-earnings",
   "weekly": [0, …], "per_history": [["2026-11-12", 23.1]], "missing": {}}]}
}
```

- 무엇이 무엇인가: 종가·지표 값·컨센서스(EPS·매출 추정, 목표주가)·외국인 보유 비중·다음 실적일은 원천(야후 파이낸스·FRED·네이버 금융·채권 종가)이 준 사실이다. 수익률·변화·52주 자리·선행 PER·EPS 수정·달러 시가총액·평균·전환선·신호·성적·괴리율·분포·베타는 그 값으로 낸 계산이다(AI 없음). `trend.rule` 의 기간·띠·비중과 괴리율의 견줄 구간·기준점 규칙은 데스크가 정한 가정이다.
- 이름이 `_pct` 로 끝나는 값과 `returns`·`delta`(`change` 가 `pct` 일 때)·`revision`·`eps.growth`·`revenue.growth`·`target.upside`·`backtest` 의 `cagr`·`mdd`·`sells` 의 평균은 % 단위의 수다(7.4 = 7.4%). `pref_gap` 의 `periods`·`anchor.gap` 도 % 다.
- 날짜가 붙은 목록은 오래된 것부터다. `trend.assets[].daily` 는 `[거래일, 종가, 평균, 전환선]`, `pref_gap.pairs[].monthly` 는 `[YYYY-MM, 월말 괴리율]`, `daily` 는 `[거래일, 괴리율, 베타만 반영한 괴리율(기준점 앞은 null)]`, `board.companies[].per_history` 는 `[날짜, 선행 PER]` 이다. `weekly` 는 최근 1년의 주간 값이고 마지막이 가장 최근이다(날짜 없이 값만).
- `trend`
  - 신호는 장이 끝난 날의 종가로 판정하고 매매는 다음 거래일 종가로 본다. `state`: `stock` | `cash`. `line` 은 종가가 넘어서면 신호가 바뀌는 값이고 `gap_pct` 는 종가의 그 값 대비다.
  - `since` 는 지금 신호가 시작된 날이고, 매매일이 아직 오지 않았으면 `trade_date`·`trade_close` 가 `null` 이다. 받아 둔 기간에 바뀐 적이 없으면 `since` 가 `null` 이다.
  - `turns` 는 최근 신호 전환(오래된 것부터 8건까지)이다. `result_pct` 는 매도(`to: cash`) 줄에만 값이 있고 그 뒤 되산 종가 ÷ 매도 종가 − 1(%)이다. 주식으로 바뀐 줄과 아직 되사지 않은 매도는 `null` 이다.
  - `backtest` 는 신호가 쌓인 기간이 한 해에 못 미치면 `null` 이고 `missing.backtest` 에 이유가 있다.
  - `backtest` 는 받아 둔 기간 전체에 같은 규칙을 적용한 성적이다. `hold` 전부 보유 · `mixed` (100 − `share_pct`)% 보유 + `share_pct`% 추세 · `trend` 전부 추세. `cash` 는 현금 쪽이 받는 금리 계열(FRED)이다.
- `macro`
  - `change` 는 `delta` 의 단위다: `bp` | `pct` | `pt`. `delta` 는 그 기간 전 날짜 이하의 가장 가까운 값과 견준 변화이고 받지 못했으면 `null` 이다.
  - `source.of`·`source.op` 이 있으면 두 지표로 계산한 값이다(`op`: 차·비율). `op` 이 `mean` 이면 `of` 의 지표들의 평균이고 `source.weights` 가 있으면 그 순서의 가중 평균이다. `bond` 는 AI 발행사 회사채 스프레드 지표에만 있다(ISIN·이표율·만기·만기수익률 `ytm`·남은 만기 `years`·종가를 낸 시장 `venue`). `cds` 는 AI 발행사 5년 CDS 지표에만 있다(가장 최근 값에 쓴 체결 수 `trades`, 그 가운데 스프레드가 적힌 체결 수 `firm` — 0 이면 선취금에서 되돌린 값 가운데 앞 날의 값에 가까운 것을 쓴 추정이다, 값이 있는 날 수 `days`, 이력의 첫 날 `since`). CDS 값은 공개 체결 보고에서 데스크가 계산한 것이고 단위는 %p 다(1%p = 100bp).
  - `home: true` 인 지표는 첫 화면의 거시 한 줄에도 보인다.
- `pref_gap`
  - 괴리율은 1 − 우선주 ÷ 보통주(%)다. `periods` 는 전체 · 견줄 구간 · 최근 1년 순서이고 `pct` 는 그 구간에서 괴리율이 지금 값 이하였던 날의 비율이다.
  - `to_median_pct` 는 보통주가 그대로일 때 괴리율이 견줄 구간(`periods[1]`)의 중앙값으로 돌아가면 우선주의 상대 수익이다. `anchor` 는 최근 `rule.anchor_days` 일 안에서 괴리율이 가장 넓었던 날과 그 뒤의 변화, `beta_gap_pct` 는 그날부터 우선주가 `beta.value` 만큼만 움직였다면의 지금 괴리율이다. 베타를 낼 수 없으면 `beta`·`beta_gap_pct` 가 함께 `null` 이다.
- `board`
  - 종목 표의 전 종목이다. `sectors` 가 화면의 묶음 순서다. 시가총액은 그 통화의 금액이고 `market_cap_usd` 는 `fx.rates` 의 환율로 나눈 달러 값이다.
  - `watch` 는 기본 관심 종목이다. 화면에서 별표로 넣고 뺀 것은 그 브라우저에만 저장되어 여기에 반영되지 않는다.
  - `per.ntm` 은 종가 ÷ 12개월 선행 EPS(이번·다음 회계연도 추정을 남은 기간으로 섞은 값)다. `revision.basis` 가 `fy1` 이면 EPS 수정이 다음 회계연도 추정 기준이다. `next_earnings.confirmed: false` 는 회사가 확정하기 전 야후가 적은 날짜다.
  - `report` 는 그 기업의 가장 최근 실적 리포트(`reports/<슬러그>.json`)이고 없으면 `null` 이다.
- `missing` 은 비워 둔 값의 이유다(키 → 이유 문장). 구획의 `missing` 은 받지 못한 지수·지표·쌍(id → 이유)이고, 줄의 `missing` 은 그 줄에서 비운 칸(필드 이름 → 이유)이다. 비운 값은 `null` 이다.

## prices.json

가격 탭(`prices/`)의 GPU 임대가격을 낸다. 값은 `content/market/prices.json` 의 기록 그대로이고 계산은 Python 이 한다(`docs/areas/prices.md`). 사이트가 내는 계산은 화면과 같은 하나(`traded.items[].vs_neocloud_pct`)뿐이다. 메모리 가격·재고는 아직 넣지 않았다. 기록이 없으면 `gpu` 가 `null` 이다.

```json
{
 "schema": 1, "updated_kst": "…",
 "gpu": {
  "unit": "USD/GPU·h", "date": "2026-11-27",
  "channels": [{"key": "hyperscaler", "label": "하이퍼스케일러 정가", "kind": "on_demand"}, {"key": "neocloud", …}, {"key": "marketplace", …}, {"key": "spot", "label": "하이퍼스케일러 스팟", "kind": "spot"}],
  "models": [{"id": "h100", "label": "H100", "marketplace_discount_pct": -23.2, "spot_discount_pct": -64.3, "spot_discount_base": 8.87}],
  "providers": [{"id": "verda", "label": "Verda", "channel": "neocloud", "note": "…", "in_reference": true, "list_basis": "min",
                 "source": {"name": "SkyPilot Catalog", "url": "…", "changed_at": "2026-11-26"}}],
  "reference": [{"model": "h100", "channel": "neocloud", "value": 3.54, "min": 2.5, "max": 5.99, "providers": 8,
                 "delta": {"d1": 0, "w1": 0, "m1": 0, "m3": 9.95, "ytd": 0, "y1": 0}, "base_date": "2023-05-07",
                 "daily": [["2025-11-30", 3.1]], "monthly": [["2023-05", 2.4]]}],
  "cells": [{"model": "h100", "provider": "verda", "kind": "on_demand", "channel": "neocloud", "median": 5.29, "min": 5.29, "rows": 1, "regions": 1,
             "date": "2026-11-27", "since": "2026-03-15", "delta": {"d1": 0, "w1": 0, "m1": 0, "m3": 0, "ytd": null},
             "range_52w": {"low": 2.29, "high": 5.62}, "weekly": [2.29, …]}],
  "regional": {"regions": [{"key": "na", "label": "북미"}], "base": "na",
               "rows": [{"model": "h100", "channel": "hyperscaler", "region": "emea", "value": 0, "min": 0, "max": 0, "providers": 2, "rows": 14, "premium_pct": 12.0}]},
  "indices": [{"id": "all", "label": "종합", "group": "all", "value": 0, "date": "…", "base_date": "…", "delta": {…}, "range_52w": {…}, "monthly": [["2022-10", 100]]}],
  "external": {"source": {"name": "Silicon Data", "url": "…"}, "values": "shape",
               "items": [{"model": "h100", "from": "…", "to": "…", "series": [{"channel": "neocloud", "shape": [0.42, null, 0.51]}]}]},
  "traded": {"source": {"name": "Ornn Data", "url": "…"}, "date": "2026-11-25",
             "items": [{"model": "h100", "index": "OCPI-H100", "gpu": "H100 SXM", "value": 3.1, "settled": "2026-11-25", "url": "…",
                        "vs_neocloud_pct": -12.4, "shape": {"from": "…", "to": "…", "values": [0.43, …]}}]},
  "terms": {"source": {"name": "GetDeploying", "url": "…", "license": "CC BY 4.0", "license_url": "…"}, "week": "2026-11-23", "min_providers": 3,
            "models": [{"id": "gb200", "label": "GB200"}],
            "rows": [{"model": "h100", "term": "r12", "label": "12개월 예약", "months": 12, "value": 3.235, "min": 1.7625, "max": 8.031,
                      "providers": 14, "offerings": 44, "since": "2025-11-24", "thin": false,
                      "delta": {"w4": 0.78, "w13": 5.92, "w52": 22.6}, "vs_on_demand_pct": -19.81, "weekly": [["2025-11-24", 2.6387]]}]}
 }
}
```

- 무엇이 무엇인가: 제공자 칸(`cells`)·계약 기간별 값(`terms.rows` 의 `value`·`min`·`max`·수)·실거래 지수의 `value` 는 원천이 준 사실이다. 채널 기준가(`reference`)·할인율·지수(`indices`)·변화율(`delta`)·`premium_pct`·`vs_on_demand_pct`·`vs_neocloud_pct` 는 그 값으로 낸 계산이다.
- 값은 GPU 한 장의 시간당 달러다. 이름이 `_pct` 로 끝나는 값과 `delta` 는 % 단위의 수다. 날짜가 붙은 목록은 오래된 것부터이고 `[날짜 또는 달, 값]` 이다. `weekly` 가 수만 든 목록(`cells`)이면 최근 52주의 주간 값이다.
- `models` 는 화면에 구획이 있는 GPU 다. `marketplace_discount_pct` 는 마켓플레이스 기준가 ÷ 네오클라우드 정가 기준가 − 1, `spot_discount_pct` 는 하이퍼스케일러 스팟 기준가 ÷ 같은 제공자의 정가 중앙값(`spot_discount_base`) − 1 이다. 견줄 값이 없으면 `null`.
- `providers[].in_reference` 가 `false` 면 기준가와 지수에 넣지 않는 제공자다. `list_basis` 는 정가 칸에서 기준가가 읽는 값(`min` 가장 싼 지역·구성 | `median` 매물의 중앙값)이고 스팟 칸은 어디서나 중앙값이다. `source.changed_at` 은 공개 가격표가 마지막으로 바뀐 날이다(있는 원천만).
- `reference` 는 모델 × 채널의 기준가다: 그 채널 제공자 값들의 중앙값(`value`)과 범위·제공자 수. `daily`(최근 1년의 기록이 있는 날)와 `monthly`(처음부터, 값이 변하지 않은 달은 빠져 있다)는 지금 기준가에 그 묶음 지수의 비율을 곱해 거슬러 올라간 달러 값이어서, 제공자가 들어오거나 빠져도 계단이 없는 대신 그때의 가격표와 조금 다를 수 있다. 실제 값은 `cells` 다.
- `cells[].channel` 은 그 칸이 드는 묶음이다. 하이퍼스케일러가 아닌 곳의 스팟은 `spot-other` 이고 기준가에 들지 않는다. `date` 가 `gpu.date` 보다 앞이면 그 뒤로 받지 못한 것이다.
- `regional.rows` 는 같은 기준가를 권역마다 따로 낸 값이고 `premium_pct` 는 기준 권역(`base`) 대비다. 권역마다 든 제공자가 달라 차이의 일부는 제공자 구성에서 온다.
- `indices` 는 여러 모델·채널을 섞은 임대가 지수(`base_date` = 100)다. `group`: `all` | `channel` | `generation` | `model`.
- `external` 은 Silicon Data 의 지수다. 원천의 약관이 값의 재게시를 막아 **값이 없다**(`values: "shape"`). `shape` 는 그림의 세로 자리(0~1)일 뿐이고 가격도 변화율도 되살릴 수 없다. 값이 없는 날은 `null`.
- `traded` 는 Ornn Compute Price Index 다(가격표가 아니라 네오클라우드에서 체결된 거래의 가격). 원천의 약관이 일별 값의 재게시를 막아 최신 정산값만 싣는다. `value` 를 쓸 때는 `index`·`settled`·`url` 을 함께 적어 인용한다. `shape` 는 `external` 과 같은 뜻이다(값 아님). `vs_neocloud_pct` 는 `value` ÷ 그 모델의 네오클라우드 정가 기준가 − 1 이다.
- `terms` 는 GetDeploying 의 주간 자료다(CC BY 4.0 — 쓸 때 `source` 의 이름과 라이선스를 밝힌다). 제공자들이 가격표에 내건 값이고 체결가가 아니다. `term`: `on_demand` | `spot` | `r<개월 수>`(예약). `value` 는 제공자마다 한 값으로 줄인 뒤의 중앙값이고, 값을 바꾼 곳이 없어도 제공자가 들어오거나 빠지면 움직인다. 제공자가 `min_providers` 보다 적은 칸은 `thin: true` 이고 `delta`·`vs_on_demand_pct` 가 `null`, `weekly` 가 없다. `terms.models` 에는 `gpu.models` 에 없는 GPU(GB200·GB300)가 있다. 끝나지 않은 주(`week`)의 값은 바뀔 수 있다.

## data/<id>/ — 조사 자료 묶음

한 질문을 두고 손으로 모은 기록이다. 화면에는 없고 feed 로만 나간다. 지금은 `apac-siting`(아시아·태평양 AI 컴퓨트 입지 비교), `ai-knowledge-work`(AI 는 코딩을 넘어 범용 지식노동으로 번지는가), `compute-balance`(누가 AI 컴퓨팅을 얼마나 갖고 쓰는가), `online-narratives`(X·Substack 의 글은 주가를 움직이는가), `open-weight-substitution`(오픈웨이트는 클로즈드 모델의 어느 일을 얼마나 가져가는가), `inference-margins`(오픈웨이트 추론을 파는 쪽에서 돈은 어디에 남는가)이다.

| 경로 | 내용 |
| --- | --- |
| `data/<id>/index.json` | 묶음별 상태와 파일 목록, 가설 판정. 가장 먼저 읽는 파일 |
| `data/<id>/<묶음>.json` | 묶음 하나의 기록 |
| `data/<id>/REQUEST.md` | 조사 요청 사본과 변경 이력 |
| `data/<id>/STATUS.md` | 묶음별 진행, 못 찾은 것, 판단하기 어려웠던 것, 값을 고친 기록 |

```json
{
 "schema": 1, "updated_kst": "…",
 "question": "한국은 아시아·태평양 AI 컴퓨트의 권역 거점이 될 수 있는가",
 "bundles": [{"id": "B", "file": "lab_overseas.json", "title": "…", "status": "collecting",
              "records": 0, "verified_share": 0.0, "updated_kst": null}],
 "hypotheses": [{"id": "H2", "text": "…", "verdict": "판단 보류", "basis": "…", "updated_kst": null}],
 "docs": ["REQUEST.md", "STATUS.md"]
}
```

- `status`: `collecting`(모으는 중) | `draft`(다 모았고 확인 전) | `verified`(큰 값을 원문으로 확인함). 시작하지 않은 묶음도 `collecting` 이고 `records` 가 0, `updated_kst` 가 `null` 이다.
- `verified_share`: 합계 가운데 원문을 열어 확인한 값의 비중(0~1). 묶음 파일의 `measure` 가 `"value"` 면 기록의 `value` 합으로, `null` 이면 건수로 낸다. 값이 없는(`null`) 기록은 값 기준 합계에서 빠진다.
- `hypotheses[].verdict`: `지지` | `반박` | `엇갈림` | `판단 보류`. 판정과 근거는 AI 의 해석이다.
- `hypotheses[].by_criterion`: 판정이 기준에 따라 갈릴 때만 있다. `[{"criterion": "…", "verdict": "…"}]`.

묶음 파일은 `{"schema", "bundle", "title", "updated_kst", "measure", "notes", "records", …}` 이다. 묶음마다 기록에서 계산한 집계(`summary`, `by_country` 등)가 더 붙을 수 있다.

기록의 공통 필드

```json
{"id": "…", "country": "KR", "city": "…", "value": 230, "unit": "MW", "asof": "2025-07-22",
 "basis": "값의 정의·기준", "value_src": [1],
 "sources": [{"n": 1, "title": "…", "url": "…", "date": "…", "read": "full"}],
 "alt": [{"value": 520, "basis": "…", "url": "…"}],
 "conf": "high", "note": "…", "revised_kst": "…"}
```

- `country` 는 두 글자 나라 코드(ISO 3166-1), `city` 는 없으면 `null`.
- `value` 는 출처가 없으면 `null` 이다. `value_src` 는 그 값을 적은 출처의 `n` 이다.
- `sources[].read`: `full`(전문을 읽음) | `partial`(해당 부분을 열어 읽음) | `search_summary`(검색 결과 요약만 봄). 값의 출처를 열어 읽지 않은 기록의 `conf` 는 `high` 가 아니다.
- `alt` 는 출처끼리 다른 값이다. 고른 이유는 `note` 에 있다.
- `revised_kst` 는 값을 고친 적이 있을 때만 있다. 고친 내용은 `STATUS.md` 에 있다.
- 용량의 기준은 `it`(IT 용량) | `utility`(IT 용량 없이 수전 용량만 공개됨) | `unknown`(기준 불명)이다. 수전 값을 IT 로 환산한 값은 넣지 않는다.

`investments.json`(묶음 A)의 기록은 발표 1건이고 대표 값은 발표 금액의 달러 값(십억 달러)이다. 공통 필드에 더해 `company`, `region`, `announced`, `amount`(`currency`·`value`·`unit`·`usd_bn`·`fx`(`rate`·`date`·`source`)·`scope`(`capex` | `capex+opex` | `unknown`)·`period`·`years`, 금액이 없으면 `null`), `mw`(`value`·`basis`·`note`), `what`(`리전 신설` | `리전 확장` | `전용 캠퍼스` | `임차`), `stage`(`발표` | `부지확보` | `착공` | `가동`), `status_now`·`status_asof`, `partners[]`, `supersedes`(이 발표가 통째로 대체한 앞 기록의 id 목록), `included_in`(금액이 더 큰 계획의 일부이면 그 계획의 id), `ref_bundle_b`(묶음 B 의 같은 건)를 갖는다. 파일의 `by_country.countries.<나라>` 에 나라별 합계가 있다: `announced_usd_bn`(발표 포함), `firm_usd_bn`(착공 이상), `announced_usd_bn_per_year`(발표 금액을 계획 햇수 `amount.years` 로 나눠 더한 연평균 — 계산값. 계획 기간이 2~10년으로 달라 함께 낸다)·`no_period`, `mw_announced`·`mw_firm`, `no_amount`, `by_company`. 대체된 기록(`by_country.superseded`)은 합계에 없고, `included_in` 이 있는 기록의 금액은 발표 합에 다시 더하지 않는다. `countries_done` 에 없는 나라는 아직 모으지 않은 것이다.

`cables.json`(묶음 C1~C3)의 기록은 해저케이블 1건이고 대표 값은 시스템 설계 용량(Tbps, 모으지 않았으면 `null`)이다. 여러 나라에 걸치므로 `country` 는 `ZZ` 이고 `countries`(닿는 대상 나라 코드), `landings[]`(대상 나라 안의 육양지), `all_countries`, `rfs`·`rfs_year`·`is_planned`, `owners[]`·`suppliers[]`, `bigtech[]`·`bigtech_mode`(`단독 소유` | `컨소시엄 구성원` | `없음`), `fiber_pairs`·`capacity`·`bigtech_pairs`(손으로 채우는 값, 없으면 `null`)를 갖는다. `by_country.<나라>` 에 케이블 수(`cables`·`planned`·`by_decade`·`since_2021`), 빅테크 참여(`with_bigtech`·`bigtech_sole`·`planned_with_bigtech`), 육양지 수, 설계 용량 합계(`design_tbps` 등, `capacity_known` 이 0 이면 아직 모으지 않은 것)가 있다. 설계 용량은 시스템 전체 값이라 나라 몫이 아니고 여러 나라에 닿는 케이블은 나라마다 전체가 잡힌다.

`peering.json`(묶음 C4·C5)의 기록은 `kind` 로 가른다. `peering` 은 도시 1곳이고 대표 값은 참여 네트워크가 있는 인터넷 교환점 수, `ixps[]`·`largest_ix_networks`·`networks_sum`·`presence.<네트워크>`(`ix`·`fac`)·`bigtech_ix_points`·`bigtech_fac_points` 를 갖는다(PeeringDB 에 사업자가 스스로 등록한 값). `latency` 는 도시 쌍 1개의 왕복 지연시간(ms, 공개 측정값을 받은 날의 값)이고 `to_country`·`to_city`·`distance_km` 를 갖는다.

`power.json`(묶음 D)의 기록은 `topic`(`D1` 품질 · `D2` 가격 · `D3` 확보 · `D4` 발전소 옆 직접 공급 · `D5` 전원 구성과 전력구매계약)과 `kind`(`reliability` | `tariff` | `access` | `onsite` | `mix` | `ppa`)로 가른다. 공통 필드에 더해 `metric`·`scope`·`definition`(정의에서 다른 나라와 달라지는 점), `key`(나라별 대표 지표의 이름), `series[]`(`period`·`value`·`src`, 해마다의 값), `currency`·`usd`(통화가 있는 값의 달러 환산 — 파일의 `fx.rates` 로 나눈 계산값), `tariff`(`price_kind`: `평균 판매단가` | `요금표` | `요금표 계산` | `도매가` | `계약가`, `includes`: 기본요금·연료비 조정·세금이 들어 있는가), `rule`(제도의 이름·기관·시행일), `onsite`(`site`·`operator`·`customer`·`fuel`·`stage`·`mw`)를 갖는다. 정전 지수와 요금은 나라마다 정의가 달라 같은 기준으로 맞추지 못했다 — `definition` 을 함께 읽어야 한다. `by_country.countries.<나라>.keys.<key>` 에 대표 지표가 모여 있다.

`demand.json`(묶음 E)의 기록은 `topic`(`E1` 인구·GDP · `E2` 도시별 데이터센터 시장 · `E3` 클라우드 시장 · `E4` AI 사용 지표 · `E5` 지연시간권)과 `kind`(`macro` | `market` | `cloud` | `ai_use` | `zone`)로 가른다. `macro` 는 나라마다 하나이고 대표 값은 지난해 명목 GDP(십억 달러), `population_m`·`gdp_per_capita_usd`·`series[]` 를 갖는다(IMF 세계경제전망). `zone` 은 거점 도시(서울·도쿄·싱가포르)와 왕복 지연시간 한도(`threshold_ms` 50·100)마다 하나이고, 대표 값은 한도 안에 닿는 나라의 인구 합(백만 명) — 나라의 대표 도시까지의 지연시간으로 판정해 그 나라 전체를 더한 계산값이며 제한 시장(중국·홍콩·러시아·미얀마)을 빼고 자기 나라를 넣었다. `gdp_usd_bn`, `excl_home`, `incl_restricted`, `any_city`, 입력인 `countries[]`(`code`·`name`·`main_city`·`rtt_ms`·`min_rtt_ms`·`reached`·`restricted`·`home`·`population_m`·`gdp_usd_bn`)를 함께 낸다. 파일의 `summary.zones` 에 여섯 값이 모여 있다.

`conditions.json`(묶음 F)의 기록은 `topic`(`F1` 재난 · `F2` 인력 · `F3` 제도 · `F4` 지정학 · `F5` 인센티브와 비용 · `F6` 기후와 용수)과 `kind`(`hazard_index` | `incident` | `talent` | `labour` | `wage` | `governance` | `data_rule` | `localisation` | `permit` | `export_control` | `conflict` | `incentive` | `build_cost` | `climate` | `water`)로 가른다. 모든 기록에 `metric`·`scope`·`definition` 이 있고 나라마다 하나인 대표 지표에는 `key` 가 붙는다(`inform_natural`, `wri_exposure`, `developers`, `ai_talent_concentration`, `wage_monthly_usd`, `rule_of_law`, `regulatory_quality`, `data_transfer`, `localisation`, `export_control`, `political_stability`, `gpi_rank`, `incentive`, `build_cost`, `water`). `auto` 가 `true` 인 기록은 공개 지수·통계에서 명령이 만든 것이다. 제도 기록(`data_rule`·`localisation`·`export_control`·`incentive`)은 값이 `null` 이고 `rule`(`name`·`authority`·`effective`·`threshold`)과 `note` 에 내용이 있으며, `export_control` 에는 `status`·`license_required`·`country_groups` 가 있다. `climate` 는 도시마다 하나이고 대표 값은 연평균 습구온도(°C), `climate` 에 건구·습구온도와 습도의 요약이 있다. 지수가 다루지 않는 나라(대만의 재난 지수 등)는 값이 `null` 이다. 채점한 값은 없다. 파일의 `by_country.countries.<나라>.keys` 에 대표 지표가, `by_country.cities` 에 도시별 기후 요약이 모여 있다. 나라 코드 `ZZ` 는 세계 전체를 다룬 기록이다.

`korea_package.json`(묶음 G)의 기록은 `topic`(`G1` 메모리·부품 · `G2` 랩과의 공개된 관계 · `G3` 국내 준비 상태)과 `kind`(`share` | `supply` | `price` | `partner` | `adoption` | `investment` | `capacity` | `program` | `law`)로 가른다. 나라는 한국과 일본·대만·인도다. `item`(무엇에 대한 기록인가), `lab`(관계된 랩이거나 `null`), `refs[]`(다른 묶음의 기록: `bundle`·`id`)가 있고, 계약·관계 기록에는 `stage`(`협의` | `발표` | `MoU` | `계약` | `착공` | `가동`)·`date`·`parties[]`(`name`·`role`)가 있다. `capacity` 는 나라마다 하나이고 값이 `null` 이며 `items[]`(`name`·`mw`·`basis`·`stage`·`online`·`ref`)에 사업자별 계획이 있다 — 기준이 섞여 있어 더하지 않는다. 파일의 `by_country.countries.<나라>` 에 `by_topic`·`by_kind`·`labs`·`stages`·`capacity_mw`(기준별 합)가 있다.

`counter.json`(묶음 H)의 기록은 묶음 A~G 에서 추린 것이다. `kind`(`against_h1` H1 에 어긋나는 사실 | `against_h2` H2 에 어긋나는 사실 | `rival_constraint` 경쟁 입지의 제약), `hypothesis`(`H1` | `H2` | `null`), `criterion`, `statement`(한 문장), `refs[]`(근거 기록: `bundle`·`id`)가 있고, 값이 있으면 `value_from`(`bundle`·`id`·`field`)이 가리키는 기록의 값과 같다. `sources[]` 는 가리키는 기록의 출처를 옮긴 것이고 `from` 에 어느 기록에서 왔는지 적혀 있다. 파일의 `summary` 에 `by_kind`·`by_hypothesis`·`by_country`·`by_criterion` 건수가 있다. 가설에 맞는 사실은 이 파일에 없다.

`gates.json`(묶음 I)의 기록은 `topic`(`I1` 대용량 전력 블록 · `I2` 돈과 임차 약정 · `I3` 수요의 종류)과 `kind`(`block` 단일 부지의 계획 용량 | `grid` 접속 심사와 기간 | `selfgen` 자가 발전과 직접 구매 | `anchor` 임차·수요 주체 | `carrier` 투자 주체의 규모 | `cost` 사업비 | `workload` 수요의 종류), `item` 을 갖는다. `block` 기록에는 `gate` 가 붙는다: `site`, `operator`, `tenant`(쓰는 쪽이거나 `null`), `mw`(`value`·`basis`), `stage`, `first_power`, `power`(전력을 받는 길), `capital`(돈을 대는 쪽), `serves`(`국내` | `권역·세계` | `불명`). 용량은 계획·목표·구상 단계의 숫자가 섞여 있어 더하지 않는다. 그 밖에 기록마다 입력 값(`detail`, `cases`)과 다른 묶음을 가리키는 `refs` 가 있다.

조사 `online-narratives` 의 묶음은 셋이다. `cases.json`(N1)의 기록은 사례 1건이고 `category`(`thesis` 장문 논지 | `relay` 유료 글·소식의 전달 | `short` 공매도 포지션 공개 | `mention` 종목 언급 | `headline` 속보 중계), `direction`(`down` | `up`), `claim`, `targets[]`(`ticker`·`name`·`role`)와 대표 종목 `primary`, `origin`(최초 글의 `author`·`platform`·`url`·`posted_at`·`precision`·`paywall`·`position_disclosed`·`reach`), `market`(`t0`·`open_at`·`close_at`·`session`), `timeline[]`(`at`·`precision`·`step`·`who`·`what`·`src`), `attribution[]`(언론이 주가를 글 탓으로 돌렸는가), `confounders[]`(같은 날의 다른 재료), `response[]`, `aftermath`(`status`·`what`), `price_as_reported[]` 를 갖는다. 대표 값은 최초 글 게시부터 다음 정규장 시작까지의 시간이다. `prices.json`(N2)의 기록은 사례 × 종목이고 `returns`(`d0`·`low0`·`high0`·`d1`·`d5`·`d20` — 전 거래일 종가 대비 누적 %), `benchmark_returns`, `excess`, `vol60`, `z0`, `retraced5`·`retraced20` 을 갖는다(아직 오지 않은 거래일은 `null`). `arrival.json`(N3)의 기록은 한국 텔레그램 채널에 처음 닿은 시각이고 `first`(`at`·`kind`·`via`), `after_close`, `hours_before_close`, `counts`, `origin_before_close` 를 갖는다. 식과 읽는 법은 파일마다의 `notes` 에, 가설 넷의 판정과 조건을 채운 사례 목록은 `index.json` 에 있다.

조사 `open-weight-substitution` 의 묶음(W1~W5: `tiers.json` 계층별 몫 · `cases.json` 특화 모델 사례 · `platforms.json` 추론·사후 학습 시장 · `labs.json` 프론티어 랩의 노출과 대응 · `counter.json` 반대 증거)의 기록은 `metric`(무엇을 잰 값인가), `definition`(누가 쟀나와 한계, 없으면 `null`), `quote`(원문 문장, 없으면 `null`), `sides`(가설 `H1`·`H2`·`H3` 마다 `지지` | `반박` | `양쪽` | `중립` — AI 의 해석), `side_basis`, `refs`, `detail` 을 더 가진다. 묶음 파일의 `summary` 에 가설별 방향 건수가 있다.

조사 `inference-margins` 의 묶음(P1~P5: `prices.json` 단가 · `costs.json` 원가 · `providers.json` 사업자 손익 · `hyperscalers.json` 하이퍼스케일러 · `counter.json` 반대 증거)의 기록은 `open-weight-substitution` 과 같은 필드를 가진다(가설 `H1`·`H2`·`H3`).

조사 `ai-knowledge-work` 의 묶음(K1~K6: `capability.json` 역량 · `usage.json` 사용 · `spend.json` 지출과 매출 · `labor.json` 노동시장 · `effect.json` 효과 측정 · `ceiling.json` 천장과 요구 수준)의 기록은 `metric`(무엇을 잰 값인가), `definition`(정의와 한계, 없으면 `null`), `links`(입증의 고리: `coding` 코딩의 존재 증명 | `spread` 코딩 밖 확산 | `ceiling` 천장 | `penetration` 침투율 | `slope` 기울기 | `counter` 반대 증거), `sides`(가설 `H1`·`H2` 마다의 방향: `지지` | `반박` | `양쪽` | `중립` — AI 의 해석), `side_basis`(그 이유)를 갖는다. 같은 지표의 시점별 값은 `series`(`period`·`value`·`src`·`label`·`note`)에, 하나로 고르지 못한 값이나 범위로 낸 계산은 `range`(`low`·`high`, 이때 `value` 는 `null` 일 수 있다)에 있다. 계산 기록은 `basis` 가 "계산"으로 시작하고 `sources` 가 비어 있으며 입력 기록을 `refs` 로 가리킨다. 묶음 파일의 `summary` 에 `by_hypothesis`·`by_link` 의 방향별 건수가 있다. `labor.json` 에서 `auto` 가 `true` 인 기록(`postings-…` 분야별 채용 공고 지수, `employment-…` 업종별 고용)은 공개 자료에서 명령으로 다시 만든 것이고, `detail` 에 기준점(2022-11) 뒤와 최근 1년의 변화(`since_base_pct`·`last_year_pct`)와 전체 대비 변화(`vs_total_…`)가 있다. `ceiling.json` 의 `auto` 기록은 임금 풀을 직접 계산한 것이다 — 입력 넷(`us-wage-bill-oews`·`world-gdp-imf`·`labor-income-share-ilo`·`office-occupation-employment-share-ilo`)과 계산 둘(`wage-pool-us-direct`·`wage-pool-global-direct`, 값은 `range` 에, 방법별 값은 `detail.us_mix_tn`·`detail.actual_mix_tn` 에 조 달러로 있다). 직종 범위와 임금 격차는 데스크의 가정이다. `index.json` 의 가설 H1 에는 고리별 판정 `by_criterion` 이 붙는다.

`lab_overseas.json`(묶음 B)의 기록은 공통 필드에 더해 `lab`, `site`, `announced`, `mw`(`value`·`basis`·`certainty`·`phases`), `stage`(`MoU` | `계약` | `착공` | `가동` — 지금까지 닿은 단계), `status_now`·`status_flag`(`진행` | `지연` | `보류` | `취소` | `불명`)·`status_asof`, `partners[]`, `package`(`capital`·`power_site`·`demand`·`components` — 파트너가 내놓은 것의 근거 문장 또는 `null`)와 `package_src`, `lab_spend`, `siting_reason[]`(`orig`·`ko`·`speaker`·`src`), `verdict`(`묶음을 따라감` | `입지를 따라감` | `둘 다` | `판단 불가` — 좁은 기준)·`verdict_basis`, `verdict_wide`·`verdict_wide_basis`(넓은 기준 — 요청의 묶음 정의 그대로), `serves`(`국내` | `권역·세계` | `불명`)·`serves_basis`, `home_country`(랩의 본국 시설이면 `true`)를 갖는다. 파일에는 판정별 집계(`summary` — `_wide` 가 붙은 것은 넓은 기준), H2 에 어긋나는 건의 묶음(`counter`), 협의 단계라 기록으로 세우지 않은 것(`pending`), 넣지 않은 것과 이유(`excluded`)가 함께 있고, 판정 기준은 `notes` 에 있다. 판정과 `serves` 는 AI 의 해석이다.

## 처음 요청과 다르게 정한 것

- **전사본**: 회사 공식본 → 거래소 공시본 → 제3자본 순서는 원칙으로 두되, 공식 전사본을 내지 않는 회사가 많아 제3자본이 기본이 된다. 전사본을 구할 수 없으면 보도자료만으로 쓰고 콜 정리 절에 "전사본 없음"이라고 적는다. 공식본 대조는 공식본이 올라온 것을 확인했을 때만 한다.
- **리포트를 만드는 계기**: 달력의 날짜가 아니라 원문이 올라온 것이다. 정기 수집이 회사 IR 의 보도자료 피드, 제3자 전사본 목록, 회사 IR 페이지의 자료 링크를 보고, 올라오면 대기 목록에 올려 하루 3편까지 리포트로 만든다. 전사본이 사흘 안에 안 나오면 보도자료만으로 먼저 내고(`status` 는 `waiting_transcript`), 나중에 전사본이 올라오면 다시 써서 채운다(`revision` 이 오른다). 달력은 확인할 기업을 줄이고 빠진 분기를 알려 주는 보조다.
- **자동으로 확인되는 범위**: 대상 56곳 중 35곳이다(보도자료와 전사본 모두 15곳, 전사본만 20곳). 미국 밖에 상장된 기업과 전사본 목록이 없는 일부 ADR 21곳은 원문을 직접 지정해야 리포트가 나온다.
- **투자자의 날**: 따로 검색하지 않고 같은 보도자료 피드와 전사본 목록에서 함께 걸러 낸다. 회사가 앞으로 열겠다고 알린 일정은 달력에 넣는다. 피드도 전사본 목록도 없는 기업의 행사는 한 달에 한 번 회사 IR 달력을 직접 읽어 달력에 넣는다.
- **날짜 확인**: 한 달에 한 번 회사 IR 달력을 직접 읽어 실적 발표일을 회사 기준으로 맞춘다(`official`). 그래서 대상 기업이라도 리포트가 없는 분기가 있다.
- **지난 일정**: 2026-11-12 이전에 발표된 실적과 투자자의 날은 채우지 않았다. 직접 지정한 것만 있다.
- **컨센서스 스냅샷**: 매출·EPS 추정만 있다. 가이던스 컨센서스는 받을 곳이 없어 넣지 않았다.
- **분기 이름**: 회사가 쓰는 이름을 따르고 `period_end` 를 함께 낸다.
- **표**: `fact_table` 에 더해 `calc_table`·`view_table` 을 두었다.
- **금융권 시각**: `judged.basis` 에 `partial` 을 더했다. 한국어 번역 제목이 있으면 `title` 에 넣고 원문 제목은 `title_orig` 에 넣었다.
- **출처**: `access` 에 `private`, `kind` 에 `presentation`·`prior_report` 를 더했다.
- **화면에서 투자자 관점의 자리**: 공통 본문 절 뒤, 출처 목록 앞에 구분된 덧붙임으로 둔다.
- **NTT DATA**: 자기 주식이 거래되지 않아 시세·컨센서스가 없다. 일정은 모회사 결산일을 본다.
- **기업별 고유 지표**: 업종별 지표(전력, 서버, 메모리, IT서비스)만 정해 두었다. 그 밖의 업종과 기업별 세부는 그 기업의 첫 리포트를 쓸 때 정한다.

## 변경 이력

### 2026-11-12 — schema 1

바뀐 것

- `feed/` 를 만들었다: `index.json`, `reports/`, `street.json`, `street/`, `calendar.json`, `health.json`.
- 실적 발표 리포트의 절 이름과 순서를 고정했다. "읽을 때 걸러 들을 점"과 "다음 분기에 확인할 지표"를 따로 두고, 투자자 관점을 공통 본문과 나눴다.
- 이미 낸 실적 리포트 2건(CoreWeave 3Q26, Bloom Energy 3Q26)을 새 틀로 다시 정리했다. 내용 판단은 바꾸지 않았다.
- 리포트 종류에 투자자의 날(`investor_day`)을 더했다.
- 실적을 따라가는 기업을 21곳에서 56곳으로 늘렸다.
- 용량 값에 기준(IT / 수전 / 기준 불명)을 붙이는 규칙과 `capacity` 필드를 더했다. 새로 쓰는 리포트부터 적용한다.
- 대상 기업의 다음 실적 발표일 33건을 달력에 넣었다.

- `calendar.json` 의 일정에 `official`(회사 IR 에서 확인한 날짜인가)을 더했다. 가까운 일정 5건(Nebius, Hon Hai, Applied Materials, Nvidia, Dell)을 회사 IR 로 확인했다.
- 투자자의 날을 보도자료 피드와 전사본 목록에서 자동으로 걸러 낸다. 첫 리포트는 Oracle Investor Day 2026 이다.
- 회사 IR 페이지의 자료 링크를 읽는 규칙을 더했다(Hon Hai 의 공식 전사본과 발표 자료, Nebius 의 보도자료).
- 보도자료만으로 먼저 낸 리포트는 전사본이 올라오면 다시 써서 채운다.
- 정기 수집에 실적 단계를 붙였다: 달력 갱신(하루 한 번), 원문 확인, 대기 목록에서 리포트 작성(하루 3편까지).

못 한 것

- 원문을 자동으로 확인하지 못하는 21곳(미국 밖 상장 기업, 전사본 목록이 없는 ADR)의 리포트. 직접 지정해야 한다.
- 발표 전 컨센서스 스냅샷의 자동 수집.
- 최근 60일 안에 대상 기업이 연 투자자의 날의 전체 목록. Oracle(10월 28일)은 찾아 리포트로 만들었지만, 피드와 전사본 목록이 없는 기업은 확인하지 못했다(`QUESTIONS.md`).
- 이미 낸 실적 리포트 2건의 공식 전사본 대조. 회사가 공식 전사본을 냈는지 확인하지 못했다(`QUESTIONS.md`).
- 야후 파이낸스에 다음 실적 발표일이 없는 기업(효성중공업, Wistron, Capgemini)의 일정.

### 2026-11-18 — schema 1

바뀐 것

- 리포트 종류에 행사 정리(`event`)를 더했다. 기술 행사·전시회·금융기관 컨퍼런스를 행사 단위로 정리한 리포트이고 절은 `s1`, `s2` … 로 나간다.
- 출처 `kind` 에 `news`(언론 보도)와 `telegram`(텔레그램 글)을 더했다. 행사 정리에서 기업 자료와 2차 자료를 가르는 데 쓴다.
- 출처 `kind` 에 `desk`(데스크 기록)를 더했다. 행사 정리가 데스크의 투자 가설을 견줄 기준으로 붙일 때 쓰고, 사실의 근거가 아니다.
- 조사 자료 묶음 `data/<id>/` 를 더했다. 첫 조사는 `apac-siting` 이다. `index.json` 에 `datasets` 를 더했다(기존 키는 그대로). 묶음마다의 기록 형식(A~H)은 위 `data/<id>/` 절에 있다.

### 2026-11-19 — schema 1

바뀐 것

- 리포트 종류에 조사 리포트(`research`)를 더했다. 조사 자료 묶음에서 쓴 중립 리포트이고 절은 `s1`, `s2` … 로 나가며 `layer` 는 모두 `common` 이다. 첫 리포트는 `apac-ai-siting-korea`(묶음 `apac-siting`)이다.
- 출처 `kind` 에 `dataset`(조사 자료 묶음 파일)을 더했다. 조사 리포트의 표가 근거로 가리킨다.
- 조사 `ai-knowledge-work`(`data/ai-knowledge-work/`)를 더했다. 묶음은 K1~K6(`capability`·`usage`·`spend`·`labor`·`effect`·`ceiling`)이고 기록의 공통 필드는 같다. 기록에 `metric`·`definition`·`links`(입증의 고리)·`sides`(가설별 방향, AI 의 해석)·`side_basis`·`series`·`range`·`refs` 가 붙고 묶음 파일에 방향별 건수 `summary` 가 있다.
- 조사 `ai-knowledge-work` 의 `labor.json`·`ceiling.json` 에 명령으로 다시 만드는 기록(`auto`)을 더했고, 가설 H1 에 고리별 판정 `by_criterion` 을 붙였다('확산' 고리 `엇갈림`).
- `QUESTIONS.md` 에서 실적·일정과 feed 일반에 관한 항목 1~16 을 지웠다(더 묻지 않기로 정해졌다). 남은 항목은 번호를 그대로 두었고(17~32), 리포트의 조건 표에 관한 33 을 더했다.
- 조사 `apac-siting` 에 묶음 I(`data/apac-siting/gates.json`, 관문 조건 — 대용량 전력 블록, 돈과 임차 약정, 수요의 종류)를 더했다. 기록의 공통 필드는 같고 단일 부지 기록에 `gate`(부지, 쓰는 쪽, 용량, 단계, 전력을 받는 길, 돈을 대는 쪽, 쓰임)가 붙는다.
- 리포트 `apac-ai-siting-korea` 를 2판으로 고쳤다(`revisions` 에 기록). 절의 수는 아홉으로 같지만 제목과 내용이 바뀌었고 표는 39개에서 17개로 줄었다 — 건별 장부는 묶음 파일에만 있다. `QUESTIONS.md` 에 34·35 를 더했다.
- 같은 리포트를 3판에서 본문과 참고 자료로 나눴다. 본문 `apac-ai-siting-korea` 는 여섯 절(답, 바닥 사실, 사는 쪽이 용량을 두는 네 갈래, 한국을 대보기, 무엇이 바뀌면 달라지나, 약한 곳)이고, 표와 계산은 새 리포트 `apac-ai-siting-korea-ref`(아홉 절)로 옮겼다. 용량 값 목록(`capacity`)은 참고 자료에만 있다. 묶음 I 에 수요의 종류 기록 셋(GPU 도입 계획의 전력 환산, Microsoft 와 Anthropic 의 용량 배치)을 더해 23건이 됐다. `QUESTIONS.md` 에 36·37 을 더했다.

### 2026-11-20 — schema 1

바뀐 것

- 조사 `online-narratives`(`data/online-narratives/`)를 더했다. 묶음은 N1 사례(`cases`) · N2 주가 반응(`prices`) · N3 한국 도착(`arrival`)이고 가설 넷(H1~H4)의 판정이 `index.json` 에 있다. 리포트는 `x-substack-price-impact` 다.
- 조사 `open-weight-substitution`(`data/open-weight-substitution/`)을 더했다. 묶음은 W1 계층별 몫(`tiers`) · W2 특화 모델 사례(`cases`) · W3 추론·사후 학습 시장(`platforms`) · W4 프론티어 랩의 노출과 대응(`labs`) · W5 반대 증거(`counter`)이고 가설 셋(H1~H3)은 모두 `판단 보류`다. 기존 키는 그대로다.

### 2026-11-23 — schema 1

바뀐 것

- `watch.json` 을 더했다. 첫 화면의 감시 대상(네 갈래의 상태·신호·그림의 값·이어진 가설·빈 곳)을 화면과 같은 계산으로 낸다. `index.json` 에 `watch` 를 더했다(기존 키는 그대로).
- `watch.json` 의 `demand` 에 `tokens`(토큰 수요의 1월 판정)를 더했다. `demand.signals` 에 신호 하나("토큰 1월 판정")가 늘었다. `tokens.rows` 에는 오픈AI 쪽(Codex PR·Homebrew·VS Code·ChatGPT 앱 순위)·구글 쪽(Gemini 코드 리뷰 PR·Gen AI SDK·Gemini CLI·Antigravity·Gemini Code Assist·Gemini 앱 순위)과 앤트로픽 쪽 계열이 짝지어 있고, 줄마다 `measure`(`volume` | `rank`)와 `vendor` 가 있다. `demand` 에 `usage`(알파벳의 모델 API 토큰·Gemini 앱 사용자, 확인용 — `demand.signals` 에는 넣지 않는다)를 더했다. 앱마다 미국 앱스토어 누적 평가 수의 주간 증가 줄(`새 평가/주`)이 있다. `demand` 에 `ramp`(기업 AI 결제 — 회사별 신호가 `demand.signals` 에 늘었다)와 `capacity`(용량 압박, 상태에 넣지 않음)를 더했고, `tokens.rows` 에 OpenRouter 결제액·Ramp 주간 지수 줄과 `group` 이 있다. 기존 키는 그대로다.
- 조사 `inference-margins`(`data/inference-margins/`)를 더했다. 묶음은 P1 단가(`prices`) · P2 원가(`costs`) · P3 사업자 손익(`providers`) · P4 하이퍼스케일러(`hyperscalers`) · P5 반대 증거(`counter`)이고 가설 셋(H1~H3)은 모두 `판단 보류`다. 기존 키는 그대로다.

### 2026-11-26 — schema 1

바뀐 것

- `market.json` 을 더했다. 시장 탭의 추세 신호·거시 지표·우선주 괴리율·종목 표를 기록 그대로 낸다. `index.json` 에 `market` 을 더했다(기존 키는 그대로).

### 2026-11-27 — schema 1

바뀐 것

- `prices.json` 을 더했다. 가격 탭의 GPU 임대가격(모델 × 채널 기준가와 추이, 제공자별 값, 지역별 값, 묶음 지수, Silicon Data 지수의 모양, Ornn 실거래 지수의 최신 정산값과 모양, GetDeploying 의 계약 기간별 주간 가격)을 기록 그대로 낸다. `index.json` 에 `prices` 를 더했다(기존 키는 그대로). 메모리 가격·재고는 아직 없다.
