# 라우트 레퍼런스

블루프린트는 세 부류로 나뉩니다. 등록은 전부 `blueprints/__init__.py: register_all()`.

1. **코어** — 로그인/메인 셸/네비게이션/공용 API
2. **업무 폼 화면** — `FORM_IDS`를 선언하고 `/portal/forms/...` prefix를 가짐. 페이지는 `GET ""`, 데이터는 `POST /api/...`. 전부 `login_required`
3. **팝업** — `FORM_IDS` 없음, `/popup/...` prefix

## 코어 라우트

| Method + Path | 파일 | 설명 |
|---|---|---|
| GET `/` | `app.py` | 로그인으로 redirect |
| GET `/portal/login` | `blueprints/login.py` | 로그인 페이지 |
| GET `/portal/logout` | `blueprints/login.py` | 세션 clear 후 로그인으로 |
| POST `/api/portal/login` | `blueprints/login.py` | 로그인 API — WCF `RequestLogin` + `CheckSystemUser` |
| GET `/portal/main` | `blueprints/main.py` | 메인 셸 (메뉴 트리) |
| POST `/api/portal/menu` | `blueprints/main.py` | 메뉴 트리 + 권한 필터 + 미구현 화면 숨김 |
| POST `/api/portal/menu_debug` | `blueprints/main.py` | 메뉴/세션 진단 덤프 |
| POST `/api/code_search` | `blueprints/main.py` | 공용 CodeService 코드 검색 (페이징) |
| POST `/api/code_validate` | `blueprints/main.py` | 코드 유효성 검증 |
| POST `/api/get_combo` | `blueprints/main.py` | 공용 CommonCode 콤보 로더 |
| GET `/portal/navigate/<form_id>` | `blueprints/__init__.py` | VB `NavigateTo` 미러 — FORM_ID → URL redirect |
| GET `/portal/chart` | `blueprints/chart.py` | 공용 분석 차트 팝업 페이지 |

## 업무 폼 화면

| FORM_ID | URL prefix | 파일 | 화면 / 주요 API |
|---|---|---|---|
| SHMBLBLLIST | `/portal/forms/bl-list` | `bl_list.py` | B/L 리스트 — `search`, `print`, `print_vsl_vyg`, `search_bl_options`, `print_guangzhou`, GET `/preview` |
| SHMBKGBOOKINGMGT | `/portal/forms/booking-mgt` | `booking_mgt.py` | 부킹 관리 — `search`, `cntr`, `location_info`, `location_nm`, `get_url`, `select_template`, `chk_trf_salesman`, `plism_link_list` |
| SPLSHMS | `/portal/forms/spl-shms` | `spl_shms.py` | 공용 검색 패널 — `search`, `schedule`, `mybl`, `custom`, `equipment`, `program` |
| SHMBLBLMGT | `/portal/forms/shm-bl-blmgt` | `shm_bl_blmgt.py` | **B/L 관리 (최대 화면, API 약 40개)** — `search`, `save`, `delete`, `get_exrate_header`, `search_hs`, `save_hs`, `apply_free_day`, `swap_eqno`, `check_tariff_valid`, `tariff_insert`, `send_plism`, `vgm_limit_chk` 등 |
| SHMSLSMONTHLYIM | `/portal/forms/sls-monthly-im` | `sls_monthly_im.py` | 월간실적회의(수입) — `search` |
| SHMSLSMONTHLYEX | `/portal/forms/sls-monthly-ex` | `sls_monthly_ex.py` | 월간실적회의(수출) — `search` |
| SHMSLSREPORTIM | `/portal/forms/sls-report-im` | `sls_report_im.py` | 업무실적회의(수입) — `auth`, `search-area`, `search-report`, `search-frt`, `search-ru`, `save-msg`, `save-sked` |
| SHMSLSREPORTEX | `/portal/forms/sls-report-ex` | `sls_report_ex.py` | 업무실적회의(수출) — `search-area`, `search-frt`, `save-sked`. **차트 팝업 기준 사례** |
| SHMSLSREPORTCN | `/portal/forms/sls-report-cn` | `sls_report_cn.py` | 업무실적회의(중국) — `search-ex-qty`, `search-ex-frt`, `search-im-qty`, `search-im-frt`. 메뉴 가상노드 |
| SHMSLSREPORTGB | `/portal/forms/sls-report-gb` | `sls_report_gb.py` | 업무실적회의(해영) — `search-vyg`, `search-report`, `bllist`, `weekinfo`. 메뉴 가상노드 |
| SHMBKGBOOKINGLIST | `/portal/forms/bkg-booking-list` | `bkg_booking_list.py` | 부킹 리스트 — `search`, `search_bkg_list`, `closing_err_check`, `save_docu_pic`, `get_dg_attach`, `get_combo` |
| SHMBKGBOOKINGLISTCN | `/portal/forms/bkg-booking-list-cn` | `bkg_booking_list_cn.py` | 부킹 리스트(중국) — `search`, `set_pickup_order`, `save`, `pickup_order_print`, `delivery_notice_print`, `print_report` |
| SHMBKGBOOKINGPRINT | `/portal/forms/shm-bkg-booking-print` | `shm_bkg_booking_print.py` | 부킹 출력 — `print_data`, `print_data_l`, `excel_data`, `get_linesoc_summary`, `get_portcostsch_list`, GET `/preview` |
| MDMCUSTMGT | `/portal/forms/mdm-cust-mgt` | `mdm_cust_mgt.py` | 고객(MDM) 관리 — `search`, `chk_other_nation`, `search_auth`, `get_state_code`, `validate_save`, `save`, `download_xls` |
| SHMBLBLPRINT | `/portal/forms/shm-bl-bl-print` | `shm_bl_bl_print.py` | B/L 출력 — `search_bl_list`, `bl_check4print`, `misu_check`, `bl_print`, `update_data`, `bl_print_log`, `upsert_sru_swb`, GET `/preview`, GET `/pdf` |
| SHMACTINVOICEMGT | `/portal/forms/shm-act-invoice-mgt` | `shm_act_invoice_mgt.py` | 회계 인보이스 관리 — `search`, `get_invoice_numbers`, `create`, `create_by_bl`, `delete`, GET `/preview` |

> API 경로는 표기 간소화를 위해 `/api/` prefix를 생략했습니다. 실제 경로는 `<URL prefix>/api/<이름>` 형태입니다 (예: `/portal/forms/bl-list/api/search`).

## 팝업 블루프린트

| URL prefix | 파일 | 화면 / 주요 API |
|---|---|---|
| `/popup/ppu_vsl_schedule` | `popup/ppu_vsl_schedule.py` | 선박 스케줄 — `get_schedule`, `get_info`, `get_route`, `get_summary`, `get_selected_schedule` |
| `/popup/ppu_bl_trf` | `popup/ppu_bl_trf.py` | B/L 운임 — `search` |
| `/popup/ppu_trf` | `popup/ppu_trf.py` | 운임 조회/선택 — `get_tariff`, `select_tariff`, `req_nomi_tariff` |
| `/popup/shm_bkg_prt_popup` | `popup/shm_bkg_prt_popup.py` | 부킹 출력 팝업 — `search_hongkong`, `print` |
| `/popup/edm_main_form` | `popup/edm_main_form.py` | EDM(전자문서/SR) — `search`, `search_ecm`, `save`, `delete`, `copy`, `create_web_sr`, `create_edi_sr`, GET `/download` |
| `/popup/shm_bl_part_bl` | `popup/shm_bl_part_bl.py` | 부분 B/L — `search` |
| `/popup/shm_bl_bl_copy` | `popup/shm_bl_bl_copy.py` | B/L 복사 — `check_bl`, `search_wharf`, `save` |
| `/popup/ppu_sch_voyage_notice_mgt` | `popup/ppu_sch_voyage_notice_mgt.py` | 항차 공지 메일 — `get_email_receiver`, `get_email_log`, `save_email_log`, `send_email` 등 |
| `/popup/shm_bl_chk_chk` | `popup/shm_bl_chk_chk.py` | B/L 체크 — `search` |
| `/popup/shm_bl_authority` | `popup/shm_bl_authority.py` | B/L 권한 — `search`, `save` |
| `/popup/shm_bl_ccam_log` | `popup/shm_bl_ccam_log.py` | CCAM 로그 — `search`, `search_detail` |
| `/popup/shm_bl_ccam_copy` | `popup/shm_bl_ccam_copy.py` | CCAM 복사 — `save` |
| `/popup/ppu_html_mail_popup` | `popup/ppu_html_mail_popup.py` | HTML 메일 발송 — `search_address`, `get_private_address`, `get_history`, `send_mail`, GET `/download` 등 |
| `/popup/shm-act-log` | `popup/shm_act_log.py` | 활동 로그 — `search` |
