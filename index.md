# PortalSNKOWeb 개발자 문서

**PortalSNKOWeb**(내부명 PortalPY)은 VB.NET WinForms로 만들어진 **Sinokor SHMS(장금상선 해운 관리 시스템)** 를 **Flask + DevExtreme** 웹으로 1:1 포팅하는 프로젝트입니다.

핵심 원칙:

- 기존 **WCF SOAP 백엔드는 그대로 재사용** — 프론트엔드/미들 레이어만 Python 웹으로 교체
- 앱은 Oracle DB에 직접 접속하지 않음 — 모든 데이터 접근은 WCF 경유
- VB 원본 코드가 동작의 진실의 원천(source of truth)

대상 사용자: 장금상선(SNKO), 흥아라인(HALK) — 한중/한일/동남아/베트남/Intra-Asia 컨테이너 해운.

## 문서 목차

| 문서 | 내용 |
|---|---|
| [시작하기](getting-started.md) | 요구사항, 설치, 서버 실행, 환경변수, 디버깅 |
| [아키텍처](architecture.md) | 전체 구조, WCF 연동, 세션 규약, 캐시 |
| [디렉토리 구조](directory-structure.md) | 폴더별 역할 |
| [라우트 레퍼런스](routes.md) | 블루프린트/엔드포인트 전체 목록 |
| [프론트엔드](frontend.md) | 템플릿, Skr 프레임워크, DevExtreme, 공용 차트 팝업 |
| [리포트 시스템](reports.md) | .repx 리포트, 변환기, .NET 렌더링 사이드카 |
| [개발 규약과 트랩](conventions.md) | 새 화면 추가 패턴, 자주 만나는 함정들 |

## 빠른 시작

```bash
pip install -r requirements.txt
python run_server.py
# → http://localhost:5837/
```

> 로그인하려면 WCF 백엔드(사내망)에 도달 가능해야 합니다. 자세한 내용은 [시작하기](getting-started.md) 참고.

## 관련 문서 (저장소 루트)

- `CLAUDE.md` — AI 작업 가이드 (아키텍처/패턴/트랩의 마스터 문서)
- `SESSION_HANDOVER.md` — 진행 중 작업 인수인계 문서
