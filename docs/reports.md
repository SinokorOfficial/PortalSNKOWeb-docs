# 리포트 시스템

리포트(B/L 출력물, 인보이스 등)는 Python이 아니라 **.NET 사이드카**가 렌더링합니다.

## 구성 요소

```
reports/            .repx/.rpt 리포트 정의 (VB Application\RPT 1:1 미러)
reports_converter/  .NET 4.8 CLI — .repx 포맷 변환기
reports_service/    .NET 8 ASP.NET Core — PDF 렌더 서비스 (포트 5838)
```

### reports/ — 리포트 정의

VB 원본 `Application\RPT`를 1:1로 미러한 폴더. 모듈별 하위 폴더:

| 폴더 | 내용 |
|---|---|
| `SHMS/` | B/L 출력 (메인, 400개+) |
| `BCMS/` | 운임/비용 |
| `BKMS/` | 부킹/인보이스 |
| `CTRT/` | 계약 |
| `EQMS/` | 컨테이너 |
| `EXPN/` | 비용 |
| `OPMS/` | 운항/스케줄 |
| `SAMS/` | 영업 |
| `Image/` | 내장 이미지 |
| `Templete/` | 공통 템플릿 |

`.repx`(DevExpress XtraReports) + `.rpt`(Crystal Reports) + `.repx.bak`이 섞여 있고 `ReportDesigner.exe`도 포함되어 있습니다.

### reports_converter/ — 포맷 변환기

- .NET 4.8 CLI (`Program.cs`)
- `.repx` **디자이너 포맷** → **XML 직렬화 포맷** 일괄 변환
- 사이드카가 `LoadLayoutFromXml`로 읽을 수 있는 형태로 만들기 위함 — 디자이너 포맷 `.repx`는 변환을 먼저 거쳐야 함

### reports_service/ — 렌더 사이드카

- .NET 8 ASP.NET Core + DevExpress XtraReports
- 주요 파일: `Program.cs`, `Controllers/ReportController.cs`, `Pages/Viewer.cshtml(.cs)`, `Reports/RepxFileStorage.cs`, `Reports/WcfClient.cs`, `appsettings.json`
- WCF 엔드포인트는 `appsettings.json`에 설정 (Flask와 동일한 백엔드)
- Flask가 iframe/proxy로 호출해 PDF를 표시

## 실행 방법

```bash
cd reports_service
dotnet run --urls=http://0.0.0.0:5838
```

요구사항:

- .NET 8 SDK
- **DevExpress 라이선스** (XtraReports v23/24/25) + DevExpress NuGet feed 등록

## 참고

- `_reports_service_src_premerge_bak/`은 병합 전 백업본 — 신규 개발과 무관 (참고용/삭제 후보)
- 상세 내용은 `reports/README.md`, `reports_service/README.md` 참고
