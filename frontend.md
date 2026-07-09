# 프론트엔드

## 스택

- **jQuery 3.6.0** (CDN)
- **DevExtreme 25.2** — 로컬 번들 `static/dx/dx.all.js` (+ `dx-license.js`). 이전 20.1.6 버전은 `static/_dx_backup_20.1.6/`에 백업
- **Skr.\*** — VB 컨트롤을 브라우저에서 재현하는 자체 프레임워크 (`static/skr/`)

## 템플릿 구조 (`templates/`)

모든 폼 화면은 `_base.html`을 extends 합니다.

- `_base.html` — head에 DevExtreme CSS(`dx.common.css`, `dx.light.css`) + `skr.css`; body 하단에 jQuery + `dx.all.js` + `dx-license.js` + Skr.* JS 로드. 헤더 색 `#1a5276`, 타이틀 "Sinokor Portal"
- `login.html`, `main.html` — 로그인 / 메인 셸
- 폼 화면 템플릿 16종 — `bl_list.html`, `booking_mgt.html`, `shm_bl_blmgt.html`, `sls_monthly_im.html` 등 (블루프린트와 1:1)
- `templates/popup/` — 팝업 14종 + `_popup_base.html`
- `chart_window.html` — 공용 분석 차트 팝업 창
- `bl_print_preview.html`, `print_preview.html` — 리포트 미리보기

## Skr 프레임워크 (`static/skr/`)

| 파일 | 역할 |
|---|---|
| `skr-base.js` | 기반 유틸 |
| `skr-enums.js` | VB enum 미러 |
| `skr-busy.js` | 로딩 모달 — `Skr.Busy.show()` 반환 객체는 **`hide()`** (`close()` 아님!) |
| `skr-dx.js` | DevExtreme 헬퍼 |
| `skr-view.js` | 화면 뷰 프레임 |
| `skr-ctl.js` / `skr-controls.js` | VB 컨트롤 재현 (텍스트박스, 콤보, 코드검색 등) |
| `skr-webgrid.js` | dxDataGrid 래퍼 (VB WebGrid 미러) |
| `skr-popup.js` | 팝업 관리 |
| `skr-chart.js` | 공용 분석 차트 엔진 (`Skr.ChartPopup`) |
| `skr-shms.js` | SHMS 도메인 공통 |

## 공용 분석 차트 팝업

어느 화면이든 버튼 하나로 별도 브라우저 창(dxPivotGrid + dxChart)을 띄우는 공용 컴포넌트입니다.

- 엔진: `static/skr/skr-chart.js` (`Skr.ChartPopup`)
- 페이지: `templates/chart_window.html`, 라우트: `GET /portal/chart` (`blueprints/chart.py`)
- **기준 사례(참고 구현): `templates/sls_report_ex.html`**
- 샘플: `static/skr/sample_im_chart.html`

새 화면에 차트 버튼을 붙일 때는 기준 사례의 호출부를 그대로 따라가면 됩니다.

## DevExtreme 관련 트랩

- `onRowPrepared`의 `e.rowElement`는 jQuery 객체일 수도, native DOM일 수도 있음 → 항상 `$(e.rowElement).css(...)` 형태로 wrap
- `format: '#,##0.00%'`는 값을 **100배** 곱해 표시 — 값이 이미 백분율 정수라면 `customizeText`를 사용
- VB의 `AllowCellMerge`(셀 병합)는 DevExtreme에 1:1 매핑이 없음 — dataSource 가공 방식으로 우회 권장
