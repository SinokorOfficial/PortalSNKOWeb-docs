# 아키텍처

## 전체 구조

```
브라우저 (jQuery + DevExtreme + Skr.*)
    │  HTTP :5837
    ▼
Flask (PortalSNKOWeb)
    │  blueprints/  ← 화면별 라우트/서버 로직
    │  wcf/         ← SOAP 클라이언트 래퍼
    │  SOAP 1.2 + WS-Addressing
    ▼
WCF BizService (사내 WCF 서버 — 주소는 wcf/client.py의 ENDPOINT_URL)
    │
    ▼
Oracle DB  (LINER. = SNKO / HLINER. = HALK 스키마)

(별도) .NET 8 리포트 사이드카 :5838 — .repx → PDF 렌더링, Flask가 iframe/proxy로 호출
```

- **Flask는 Oracle에 직접 접속하지 않습니다.** 모든 쿼리/프로시저 실행은 WCF SOAP을 경유합니다.
- VB 원본(WinForms)의 화면 1개가 Flask 블루프린트 1개로 매핑됩니다.

## WCF 연동 (`wcf/`)

- 코어 클라이언트: `wcf/client.py` — 엔드포인트 상수 `ENDPOINT_URL`, wsHttpBinding, SOAP 1.2 + WS-Addressing
- 기본 호출 메서드는 `MethodCallJsonStream` — 응답이 gzip+base64 압축되어 대용량에서 ~70% 절감
- `requests.Session` 커넥션 풀을 재사용
- 도메인별 래퍼 파일 약 20개: `auth.py`(로그인/메뉴), `procedure.py`(프로시저), `code.py`(코드서비스), `bl.py`, `booking.py`, `tariff.py`, `edm.py` 등 — 화면 하나를 만들 때 래퍼 파일 하나를 추가하는 구조

### Oracle 저장 프로시저 호출

`wcf/procedure.py: call_procedure()` — VB `SamsAPI.Data.SamsData.CallProcedure`의 미러. 2단계로 동작:

1. `QryPrcParam`으로 프로시저 파라미터 메타 조회
2. `(Proc)<name>` + `ProcParameter` 테이블을 구성해 `QryPrcData`로 실행

LineCode에 따라 스키마 prefix가 결정됩니다 (`SCHEMA_BY_LINECODE`):

| LineCode | 스키마 |
|---|---|
| `SNKO` | `LINER.` |
| `HALK` | `HLINER.` |

### 응답 형태 규약

- 응답 테이블 키는 **대문자 + `DATA.` prefix** (예: `DATA.PRESULTSETAREA`) — 래퍼가 mixed-case alias(`pResultsetArea`)를 추가해 줌
- 상태 체크: `tables["KEY_VALUE_PAIRS"][0]["STATUS"] == "SUCCESS"`

## 세션 규약 (★ 자주 틀리는 부분)

로그인 시 WCF `RequestLogin` + `CheckSystemUser` 응답으로 세션을 채웁니다 (`blueprints/login.py`).

| 세션 키 | 값 형태 | 의미 |
|---|---|---|
| `wcf_user_id` | 회사 이메일 | 로그인 이메일 = WCF 인증 uid |
| `wcf_usr_id` | 사번 | **사번** (≤10자). Oracle 프로시저 `pEmpno`에는 반드시 이 값 |
| `wcf_compcd` | `SNKO` 등 | 회사코드 |
| `wcf_line_code` | `SNKO` 등 | LineCode — Oracle 스키마 결정 |
| 기타 | | `wcf_auth_agc_cd`, `wcf_auth_la_list`, `wcf_sales_lvl`, `wcf_office_cd`, `wcf_ctry_cd`, `wcf_curr_cd`, `wcf_eng_nm`, `wcf_email`, `wcf_office_nm` |

> **트랩**: `pEmpno`에 이메일(20자+)을 넣으면 Oracle `EMPNO VARCHAR2(10)`에서 **ORA-12899**가 발생합니다. 반드시 사번(`wcf_usr_id`)을 사용하세요.

- 세션에는 WCF 자격증명(이메일 + 비밀번호)이 보관되며 WCF 호출 시 `creds()`로 꺼내 사용
- 공용 세션/캐시/인증 유틸: `blueprints/_common.py` (`session_ctx`, `login_required` 등)

## 캐시

`blueprints/_common.py`의 `_WCFCache` — 인메모리 LRU + TTL:

- 기본 200개 항목 / TTL 60초
- 빈 결과·에러 응답은 캐시하지 않음
- Oracle 응답이 최대 ~70초 걸릴 수 있어 캐시 적중 시 체감 성능 차이가 큼
- 개발 중 캐시 때문에 헷갈리면 `PORTAL_NO_CACHE=1`

## 메뉴 / 권한 (`blueprints/main.py`)

- WCF `RetrieveMenu` + `SynchronizeUserSessionInfo`로 메뉴 트리 구성
- ADMIN은 전체 메뉴, 그 외는 권한 프로그램ID로 필터
- `PORTAL_HIDE_UNIMPL`(기본 ON)이 미포팅 화면(FORM_URL_MAP에 없는 leaf)을 숨김
- 실적회의 중국/해영(CN/GB) 화면은 수입(IM) 형제 노드로 가상 주입(`_inject_cn_menu`)

## 화면 네비게이션

VB의 `MainFormShms.NavigateTo(SrcID)`를 미러:

- 각 블루프린트가 `FORM_IDS = ["VB_FORM_ID"]`를 선언하면 `blueprints/__init__.py`가 자동으로 `FORM_ID → url_prefix` 매핑을 생성
- `GET /portal/navigate/<form_id>` → 매핑된 URL로 redirect (미매핑 시 404)
