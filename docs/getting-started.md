# 시작하기

## 요구사항

- **Python 3.12**
- 의존성은 단 3개: `flask>=3.0`, `requests>=2.31`, `waitress>=3.0`
- Oracle 드라이버 불필요 — DB 접근은 전부 WCF SOAP 경유
- WCF 백엔드(사내망 BizService 엔드포인트 — 주소는 `wcf/client.py`의 `ENDPOINT_URL` 참고)에 도달 가능해야 로그인/데이터 조회 가능
- (선택) 리포트 PDF 렌더링: .NET 8 SDK + DevExpress 라이선스 → [리포트 시스템](reports.md) 참고

## 설치

```bash
git clone https://github.com/SinokorOfficial/PortalSNKOWeb.git
cd PortalSNKOWeb
pip install -r requirements.txt
```

> 소스 저장소는 private입니다. IT-Team 소속 GitHub 계정으로 접근하세요.

## 서버 실행

진입점은 4개이며 전부 `app.create_app()` 팩토리를 사용하고 기본 포트는 **5837**입니다.

| 파일 | 용도 | 서버 | 비고 |
|---|---|---|---|
| `run_server.py` | **개발용 (권장)** | Werkzeug | `debug=True`, reloader 꺼짐(자식 프로세스가 5837 포트를 두고 부모와 경쟁해 로그인 hang이 발생하는 문제 방지) |
| `serve.py` | **운영용 (권장)** | Waitress | `logs/portal.log` 로테이팅 로그(10MB×10) 포함. NSSM으로 Windows 서비스 등록 상정 |
| `run_waitress.py` | 간이 운영 | Waitress | 로깅 설정 없음 |
| `app.py` | 직접 실행도 가능 | Werkzeug | 주 용도는 app factory 정의 |

```bash
# 개발
python run_server.py

# 운영
python serve.py
```

접속: `http://localhost:5837/` → 자동으로 `/portal/login`으로 이동합니다.
로그인 계정은 이메일 + 비밀번호 체계이며 WCF `RequestLogin`으로 인증됩니다.

## 환경변수

| 변수 | 기본값 | 효과 |
|---|---|---|
| `PORTAL_SECRET` | (없음) | Flask 세션 서명 키. 미설정 시 `.flask_secret` 파일 사용/생성. **운영에서는 이 변수로 지정 권장** |
| `PORTAL_PORT` / `PORT` | `5837` | 바인딩 포트 |
| `PORTAL_HOST` | `0.0.0.0` | 바인딩 호스트 |
| `PORTAL_THREADS` / `THREADS` | `8` | Waitress 스레드 수 |
| `PORTAL_NO_CACHE` | `0` | `1`이면 인메모리 WCF 캐시 비활성 (개발 시 유용) |
| `PORTAL_HIDE_UNIMPL` | `1` | 미포팅 화면을 메뉴에서 숨김. `0`이면 전부 표시 |
| `WCF_DEBUG` | `0` | `1`이면 `_dump/_last_request.xml`·`_last_response.xml` 저장 + `[wcf-debug]` 로그 (느려짐) |
| `PROC_DEBUG` | `0` | `1`이면 `[proc]` CallProcedure 상세 로그 |
| `WCF_TRX_LOG` / `WCF_TIMING` | `0` | TrxCode / 응답시간 로그 |

## 세션 키 관리

- 세션 서명 키 우선순위: `PORTAL_SECRET` 환경변수 → `.flask_secret` 파일 → 신규 생성 후 파일 저장
- 실행마다 키가 바뀌면 세션 쿠키 서명이 무효화되어 간헐적 로그인 튕김이 발생하므로 키를 영속화함
- 세션 수명 12시간, rolling 갱신(`SESSION_REFRESH_EACH_REQUEST=True`), `SameSite=Lax`
- 현재 HTTP 운영 전제라 `SESSION_COOKIE_SECURE` 미설정 — **HTTPS 전환 시 `app.py`에서 True로 변경 필요**

## 디버깅 팁

- `.claude/launch.json` 디버그 프로필은 `PROC_DEBUG=1, WCF_DEBUG=1, PORTAL_NO_CACHE=1`로 `run_server.py`를 기동
- `WCF_DEBUG=1`로 실행하면 `_dump/` 아래에 마지막 SOAP 요청/응답 XML이 저장됨 (`_dump/`는 gitignore 대상)
- Oracle 응답이 대용량(8000행+)이면 최대 ~70초 걸릴 수 있음 — 캐시 적중 시 즉시 반환. [아키텍처](architecture.md)의 캐시 절 참고
