# 디렉토리 구조

```
PortalSNKOWeb/
├── app.py                 # Flask app factory (create_app)
├── run_server.py          # 개발 서버 (Werkzeug)
├── serve.py               # 운영 서버 (Waitress + 로깅) ← 운영 권장
├── run_waitress.py        # 간이 운영 서버
├── requirements.txt       # flask, requests, waitress — 단 3개
├── .flask_secret          # 세션 서명 키 (gitignore 대상)
├── CLAUDE.md              # AI 작업 가이드 (마스터 문서)
├── SESSION_HANDOVER.md    # 진행 중 작업 인수인계
├── .claude/               # 디버그 실행 설정, 권한 allowlist
│
├── blueprints/            # ★ 화면 1개 = 파일 1개 (라우트/서버 로직)
│   ├── __init__.py        #   register_all() + FORM_ID → URL 자동 매핑
│   ├── _common.py         #   세션/캐시/인증 공용 유틸
│   └── popup/             #   팝업 전용 블루프린트
│
├── wcf/                   # ★ WCF SOAP 래퍼 (도메인별 1파일, 약 20개)
│   ├── client.py          #   SOAP 코어 클라이언트
│   ├── procedure.py       #   Oracle 프로시저 호출 (2단계 SOAP)
│   └── auth.py, bl.py, booking.py, tariff.py, ...
│
├── templates/             # Jinja2 템플릿 (모든 폼은 _base.html extends)
│   └── popup/             #   팝업 화면 템플릿 (14종 + _popup_base.html)
│
├── static/
│   ├── dx/                # DevExtreme 25.2 번들 (dx.all.js 등)
│   ├── _dx_backup_20.1.6/ # 이전 버전 백업
│   ├── skr/               # ★ 자체 프론트 프레임워크 (Skr.* — VB 컨트롤 재현)
│   └── css/, js/, popup/
│
├── reports/               # VB Application\RPT 1:1 미러 — .repx/.rpt 리포트 정의
│   ├── SHMS/              #   B/L 출력 (메인, 400개+)
│   ├── BCMS/, BKMS/, CTRT/, EQMS/, EXPN/, OPMS/, SAMS/
│   ├── Image/             #   내장 이미지
│   └── Templete/          #   공통 템플릿
│
├── reports_converter/     # .NET 4.8 CLI — .repx 디자이너 포맷 → XML 직렬화 포맷 변환
├── reports_service/       # ★ .NET 8 사이드카 — .repx → PDF 렌더 (포트 5838)
├── _reports_service_src_premerge_bak/  # 병합 전 백업 (사문서 — 참고용)
│
└── shipping-ontology/     # 해운 도메인 온톨로지 YAML (AI/RAG용 — 앱 런타임과 무관)
    ├── 00_domain_config.yaml, 01_glossary.yaml
    ├── entities/          #   bill_of_lading, booking, container_move_history 등
    ├── relations/         #   booking_to_port_dg, vessel_to_route 등
    └── scenarios/         #   booking, dg_handling, schedule_change
```

## 주의사항

- `*.premerge_bak` 파일들(블루프린트/템플릿/JS)은 병합 전 백업으로 **라우트에 등록되지 않으며 수정 대상이 아닙니다**
- `_dump/`는 WCF 디버그 덤프 폴더로 gitignore 대상 — 클론 직후에는 없을 수 있음
- `shipping-ontology/`는 앱 코드가 로드하지 않는 문서성 YAML입니다
