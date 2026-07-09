# 개발 규약과 트랩

## 새 화면 추가 패턴 (★ 핵심)

VB 폼 1개 = **4개 파일 + 1줄 편집**:

1. `wcf/<name>.py` — WCF 래퍼 (`call_procedure` 사용)
2. `blueprints/<name>.py` — 라우트/서버 로직. **`FORM_IDS = ["VB_FORM_ID"]` 선언** (대문자 VB 폼ID)
3. `templates/<name>.html` — {% raw %}`{% extends "_base.html" %}`{% endraw %}
4. `blueprints/__init__.py`의 `_FORM_MODULES` 리스트에 모듈 한 줄 추가

`FORM_IDS`만 선언하면 `/portal/navigate/<form_id>` → url_prefix 매핑이 자동 생성됩니다 (VB `NavigateTo` 미러).

**참고용 모범 템플릿: `blueprints/bl_list.py`, `blueprints/sls_monthly_im.py`**

## VB 원본 확인

동작이 의심스러우면 VB 원본을 확인하는 것이 원칙입니다 (진실의 원천):

- `D:\SinokorPortalV2\Application\` 또는 `D:\ProgramSource\SinokorPortalV2\`

## 자주 만나는 트랩

1. **`Skr.Busy`** — `Skr.Busy.show()` 반환 객체의 메서드는 `hide()`입니다. `close()`가 아님.
2. **`onRowPrepared`의 `e.rowElement`** — jQuery/native DOM이 혼재 → 반드시 `$(e.rowElement)`로 wrap.
3. **`pEmpno`에는 사번** — 이메일(20자+)을 넣으면 `EMPNO VARCHAR2(10)`에서 ORA-12899. 세션의 `wcf_usr_id`(사번)를 사용.
4. **Oracle 응답 지연** — 대용량(8000행+)이면 최대 ~70초. 캐시 적중 시 즉시 반환되므로 성능 판단 시 주의.
5. **`format: '#,##0.00%'`** — 값을 100배 곱함. 이미 백분율 정수면 `customizeText` 사용.
6. **VB `AllowCellMerge`** — DevExtreme 1:1 매핑 없음. dataSource 가공으로 우회.
7. **IN 파라미터 빈값 정책** — VB는 `""`(빈 문자열)를 자주 보냄. `null`을 보내면 PL/SQL의 `IS NULL` 분기가 달라질 수 있음.
8. **응답 테이블 키** — 대문자 + `DATA.` prefix (`DATA.PRESULTSETAREA`). 래퍼가 mixed-case alias(`pResultsetArea`)를 추가. 상태 체크는 `tables["KEY_VALUE_PAIRS"][0]["STATUS"] == "SUCCESS"`.
9. **reloader 금지** — Werkzeug reloader를 켜면 자식 프로세스가 5837 포트를 두고 부모와 경쟁해 로그인이 hang. `run_server.py`는 이미 `use_reloader=False`.

## 문서화 규칙

- `CLAUDE.md` 규칙: **AGENTS.md/CLAUDE.md 외의 .md 문서를 임의로 만들지 말 것.** 인수인계가 필요할 때만 `SESSION_HANDOVER.md`를 갱신
- 이 `docs/` 폴더는 팀 공유용 개발 문서로 예외적으로 유지 — 새 화면/규약이 추가되면 함께 갱신해 주세요

## 백업 파일 정책

- `*.premerge_bak` (블루프린트/템플릿/JS), `_reports_service_src_premerge_bak/`, `static/_dx_backup_20.1.6/` — 전부 병합/업그레이드 전 백업. **수정·참조 대상이 아닙니다**
