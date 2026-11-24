# feed 형식 명세

이 사이트의 리포트와 실적 일정을 정적 JSON 파일로 내보낸다. JavaScript 실행이나 로그인 없이 읽을 수 있다.
사이트를 빌드할 때 화면과 같은 원천에서 함께 만들며, 손으로 따로 관리하지 않는다. 자료는 이 사이트에서 밖으로만 나간다.

- 기준 주소: `<사이트 주소>/feed/`
- 형식 번호: `schema` = 1 (모든 JSON 파일의 맨 위에 있다)

## 파일

| 경로 | 내용 |
| --- | --- |
| `index.json` | 목록. 가장 먼저 읽는 파일 |
| `reports/<슬러그>.json` | 리포트 1건(실적 발표·투자자의 날·인터뷰·글 분석·행사 정리·종합·조사 리포트). `config/site.toml`의 `[sealed]`에 적은 유형은 싣지 않는다(지금은 없음) |
| `calendar.json` | 실적·투자자의 날 일정과 대기 목록 |
| `health.json` | 수집 상태와 놓친 것(`missed`) |
| `SPEC.md` | 이 문서 |
| `QUESTIONS.md` | 판단하기 어려웠던 것 |

증권 리포트 목록과 지표는 내보내지 않는다.

## 파일 규칙

- UTF-8(BOM 없음), 줄바꿈 LF. 한 파일 1.5MB 이하.
- 시각은 한국시간이다(`2026-11-12T09:12:00+09:00`). 날짜만 있는 필드는 `YYYY-MM-DD`.
- 값이 없는 선택 필드는 키를 넣지 않는다. `null` 을 쓰는 필드는 아래에 따로 적었다.
- 리포트를 고치면 `revision` 이 1 오르고 `revised_kst` 와 `review.history` 에 남는다. 조용히 바꾸지 않는다.
- 필드 이름은 바꾸지 않는다. 바꿔야 하면 `schema` 번호를 올리고 이 문서의 변경 이력에 적는다. 값의 종류(예: 출처 `kind`)는 늘어날 수 있다.
- 유료 출처(`access: "paid"`)는 화면과 똑같이 본문을 싣지 않는다. 짧은 인용과 이 사이트의 문장으로 쓴 사실·해석만 나간다. 화면에 없는 것을 feed 에 더 넣지 않는다.
- 이미지·영상은 파일로 넣지 않는다. 원본 주소와 시각(초, `t`)만 넣는다.
- 게시 전에 화면의 리포트 수가 feed 의 수와 같은지 검사한다. 다르면 게시하지 않는다.

## index.json

```json
{
 "schema": 1,
 "generated_kst": "2026-11-12T09:16:31+09:00",
 "reports": [
  {"slug": "sysco-1q27-earnings", "type": "earnings",
   "title": "…", "companies": [{"name": "Sysco", "ticker": "SYY"}],
   "event": "1Q27 실적 발표", "event_date": "2026-11-03",
   "published_kst": "…", "revised_kst": "…", "revision": 3,
   "summary": "목록에 보이는 요약(해석)", "path": "reports/sysco-1q27-earnings.json"}
 ],
 "calendar": {"updated_kst": "…", "path": "calendar.json"}
}
```

- `generated_kst` 는 빌드 시각이 아니라 **가장 최근에 바뀐 기록의 시각**이다. 내용이 그대로면 이 값도 그대로다.
- `type`: `earnings` | `investor_day` | `interview` | `article` | `event`(행사 정리) | `synthesis`(종합) | `research`(조사 리포트).
- `companies[].ticker` 는 없으면 `null`. `event`·`event_date` 는 있을 때만.
- `revision` 은 처음 게시할 때 1이다. `revised_kst` 는 고친 적이 없으면 `published_kst` 와 같다.
- `path` 는 `feed/` 기준 상대 경로다.

## reports/<슬러그>.json

```json
{
 "schema": 1, "slug": "…", "type": "earnings", "title": "…", "summary": "…",
 "companies": [{"name": "Sysco", "ticker": "SYY"}], "event": "…", "event_date": "…",
 "period": "3Q26", "period_end": "2026-09-30",
 "prev_slug": null,
 "published_kst": "…", "revised_kst": "…", "revision": 3,
 "sources": [
  {"n": 1, "title": "Sysco 1Q27 실적 보도자료", "url": "…", "date": "2026-11-03",
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
 "watch_next": ["다음 분기에 확인할 지표 문장", "…"],
 "review": {"draft": {"model": "…", "at_kst": "…"}, "review": {"model": "…", "at_kst": "…", "result": "통과"},
            "summary": "…", "history": [{"at_kst": "…", "text": "…"}]}
}
```

### 두 겹: `layer`

- `common` — 공통 본문. 누구의 관점도 넣지 않은 중립 본문이다. 사실·발언·계산과, 원문에서 바로 따라 나오는 해석까지.
- `trade` — 현업 시사점. 한국의 식자재 유통·급식 사업에 주는 뜻(원가·판가·경쟁·고객 업종). 공통 본문 뒤의 덧붙임이며 파일에는 표시만 해 두고 넣는다. 받아 쓰는 쪽이 골라 쓴다.

### 절

실적 발표와 투자자의 날은 절의 `id`·이름·순서가 고정이다. 마지막 `trade_view` 만 없을 수 있다.

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
| `trade_view` | 현업 시사점 | 현업 시사점 | trade |

- 전사본이 없어 콜 정리를 싣지 못한 리포트도 `prepared`·`qa` 절은 있다. 그 사실을 `view` 블록 하나로 적는다.
- 인터뷰·글 분석·행사 정리는 화면의 절을 그대로 내보낸다. `id` 는 순번(`s1`, `s2` …)이고 `layer` 는 모두 `common` 이다.
- 종합·조사 리포트(`synthesis`·`research`)도 화면의 절을 그대로 내보낸다. `config/site.toml` 의 `[sealed]` 에 적은 유형만 `feed/` 에 싣지 않는다(지금은 없음).
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
- `sources[].kind`: `press_release` | `transcript_official` | `transcript_third_party` | `presentation` | `market_data` | `prior_report` | `article` | `video` | `news`(언론 보도) | `dataset`(데스크가 모아 둔 자료 표) | `filing`(규제 공시·정기보고서). 인터뷰·글 분석은 `video` 또는 `article` 이다.
- `sources[].read`: 실제로 읽은 범위. `전문` | `일부` | `요약만`.
- `period` 는 회사가 쓰는 분기 이름이다(회계연도가 달력과 다른 회사 포함). `period_end` 는 그 분기의 말일이라 회사끼리 견줄 때 쓴다. 둘 다 있을 때만 들어간다.
- `prev_slug`: 이 사이트에 직전 분기 리포트가 있으면 그 슬러그, 없으면 `null`.
- `watch_next`: `watch_next` 절의 항목을 문자열 배열로 다시 낸 것이다. 그 절이 있는 리포트에만 있다.
- `review.summary` 는 검토 요약이 없으면 `null`.

## calendar.json

```json
{
 "schema": 1, "updated_kst": "…",
 "events": [{"company": "Sysco", "ticker": "SYY", "type": "earnings", "period": null,
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
                {"name": "calendar", "items": 33, "last_kst": "…"}],
 "missed": [{"company": "…", "ticker": "…", "type": "earnings", "period": null, "date": "…", "reason": "…"}]
}
```

- `collectors[].last_kst` 는 그 기록이 마지막으로 바뀐 시각이다. 오래 멈춰 있으면 수집이 돌지 않은 것이다.
- `missed` 는 `calendar.json` 에서 `status` 가 `missed` 인 일정이다.

## 변경 이력

### 2026-11-21 — schema 1

- 이 데스크의 `feed/` 를 시작한다: `index.json`, `reports/`, `calendar.json`, `health.json`.
- 실적 발표·투자자의 날의 마지막 절은 `trade_view`(현업 시사점)이고 `layer` 는 `trade` 다.
