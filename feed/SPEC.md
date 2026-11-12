# feed 형식 명세

이 사이트의 리포트와 금융권 시각을 정적 JSON 파일로 내보낸다. JavaScript 실행이나 로그인 없이 읽을 수 있다.
사이트를 빌드할 때 화면과 같은 원천에서 함께 만들며, 손으로 따로 관리하지 않는다. 자료는 이 사이트에서 밖으로만 나간다.

- 기준 주소: `<사이트 주소>/feed/`
- 형식 번호: `schema` = 1 (모든 JSON 파일의 맨 위에 있다)

## 파일

| 경로 | 내용 |
| --- | --- |
| `index.json` | 목록. 가장 먼저 읽는 파일 |
| `reports/<슬러그>.json` | 리포트 1건(실적 발표·투자자의 날·인터뷰·글 분석) |
| `street.json` | 금융권 시각 최근 30일 항목과 지수 |
| `street/YYYY-MM.json` | 금융권 시각 월별 보관(한국시간 공개 월) |
| `calendar.json` | 실적·투자자의 날 일정과 대기 목록 |
| `health.json` | 수집 상태와 놓친 것(`missed`) |
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
- 게시 전에 화면의 리포트 수·금융권 항목 수가 feed 의 수와 같은지 검사한다. 다르면 게시하지 않는다.

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
 "calendar": {"updated_kst": "…", "path": "calendar.json"}
}
```

- `generated_kst` 는 빌드 시각이 아니라 **가장 최근에 바뀐 기록의 시각**이다. 내용이 그대로면 이 값도 그대로다.
- `type`: `earnings` | `investor_day` | `interview` | `article` | `synthesis`.
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
- 인터뷰·글 분석·종합은 화면의 절을 그대로 내보낸다. `id` 는 순번(`s1`, `s2` …)이고 `layer` 는 모두 `common` 이다.
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
| `calc` | 가정과 사실로 계산한 값 | `text`, `inputs[]`(`name`, `value`), `formula` |
| `assumption` | 계산·판단의 전제 | `by`(`ai` \| `user`), `text` |
| `explain` | 배경 설명(AI) | `text` 또는 `items` |
| `view` | 해석·의견(AI) | `text` 또는 `items` |

- `src` 는 `sources[].n` 이다. `loc` 은 원문 안 위치(쪽, 소제목, `Q&A 3`, 영상 시각 `[12:34]`)다.
- `t` 는 `loc` 에 영상 시각이 있을 때 그 시작 시각(초)이다.
- `text` 는 빈 줄로 문단을 나누고 강조는 `**굵게**` 만 쓴다.

### 그 밖의 필드

- `sources[].access`: `public` | `paid` | `private`(공개 링크가 없는 출처. `url` 이 없다).
- `sources[].kind`: `press_release` | `transcript_official` | `transcript_third_party` | `presentation` | `market_data` | `prior_report` | `article` | `video`. 종류가 적혀 있지 않은 인터뷰·글 분석의 출처는 주소가 유튜브면 `video`, 아니면 `article` 로 낸다.
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

## 처음 요청과 다르게 정한 것

- **전사본**: 회사 공식본 → 거래소 공시본 → 제3자본 순서는 원칙으로 두되, 공식 전사본을 내지 않는 회사가 많아 제3자본이 기본이 된다. 전사본을 구할 수 없으면 보도자료만으로 쓰고 콜 정리 절에 "전사본 없음"이라고 적는다. 공식본 대조는 공식본이 올라온 것을 확인했을 때만 한다.
- **리포트를 만드는 계기**: 달력의 날짜가 아니라 원문이 올라온 것이다. 정기 수집이 회사 IR 의 보도자료 피드와 제3자 전사본 목록을 보고, 올라오면 대기 목록에 올려 하루 3편까지 리포트로 만든다. 전사본이 사흘 안에 안 나오면 보도자료만으로 먼저 낸다(`status` 는 `waiting_transcript`). 달력은 확인할 기업을 줄이고 빠진 분기를 알려 주는 보조다.
- **자동으로 확인되는 범위**: 대상 56곳 중 34곳이다(보도자료와 전사본 모두 13곳, 전사본만 21곳). 미국 밖에 상장된 기업과 전사본 목록이 없는 일부 ADR 22곳은 원문을 직접 지정해야 리포트가 나온다. 그래서 대상 기업이라도 리포트가 없는 분기가 있다.
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
- 정기 수집에 실적 단계를 붙였다: 달력 갱신(하루 한 번), 원문 확인, 대기 목록에서 리포트 작성(하루 3편까지).

못 한 것

- 원문을 자동으로 확인하지 못하는 22곳(미국 밖 상장 기업, 전사본 목록이 없는 ADR)의 리포트. 직접 지정해야 한다.
- 발표 전 컨센서스 스냅샷의 자동 수집.
- 투자자의 날 리포트. 아직 한 건도 없다.
- 최근 60일 안에 대상 기업이 연 투자자의 날 목록. 뉴스 검색이 최근 며칠치만 돌려줘 확인하지 못했다(`QUESTIONS.md`).
- 이미 낸 실적 리포트 2건의 공식 전사본 대조. 회사가 공식 전사본을 냈는지 확인하지 못했다(`QUESTIONS.md`).
- 야후 파이낸스에 다음 실적 발표일이 없는 기업(효성중공업, Wistron, Capgemini)의 일정.
