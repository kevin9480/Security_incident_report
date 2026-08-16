# 05. 침해 지표 (IOC)

침해사고 분석 보고서 4장을 기반으로 정리한 침해 지표입니다. 실제 악성 바이너리/페이로드는 공개하지 않고, 탐지·차단에 쓸 수 있는 지표만 정리했습니다.

## 4.1 주요 IOC 요약

| 구분 | 주요 지표 | 설명 |
|---|---|---|
| 공격자 IP | `192.168.202.94` | DMZ Web 서버 최초 접근 및 악성 파일 다운로드 서버 |
| 침해 서버 | `10.10.10.2` | RCE가 수행된 DMZ Web 서버 |
| DB 서버 | `10.10.20.3` | DMZ Web 서버에서 접근한 MongoDB 서버 |
| 감염 PC | `10.10.30.2` | 악성 첨부파일 실행 및 랜섬웨어 행위가 확인된 최초 감염 PC |
| 확산 대상 PC | `10.10.30.5` | WinRM 기반 원격 실행 및 후속 감염이 확인된 PC |
| 주요 악성 파일 | `poster.zip` 외 | 사용자 감염, 후속 페이로드 실행, 랜섬웨어 실행 관련 파일 |
| 주요 계정 | `internal_app`, `webadmin` | DB 인증 계정 및 웹 서버 명령 실행 계정 |
| 주요 포트 | `5985/TCP`, `8082/TCP` | WinRM 내부 확산 및 악성 파일 다운로드에 사용된 포트 |

## 4.2 네트워크 및 접속 지표

| 지표 | 방향 | 의미 | 대응 활용 |
|---|---|---|---|
| `192.168.202.94 → 10.10.10.2` | 외부 공격자 → DMZ Web 서버 | 최초 HTTP 접근 및 RCE 시도 | 방화벽/WAF/웹 로그 기반 차단 및 헌팅 |
| `10.10.10.2 → 10.10.20.3` | DMZ Web 서버 → DB 서버 | MongoDB 접속 및 인증 | DB 접근 제어 정책 점검 |
| `10.10.30.2 → 10.10.30.5:5985` | 최초 감염 PC → 확산 대상 PC | WinRM 기반 내부 확산 | 내부망 방화벽 및 EDR 탐지 |
| `192.168.202.94:8082` | 사용자 PC → 공격자 서버 | `downloader.exe` 다운로드 경로 | 프록시/방화벽/DNS·HTTP 로그 차단 |

## 4.3 파일 및 경로 지표

| 파일명 또는 경로 | 판단 | 설명 |
|---|---|---|
| `poster.zip` | 악성 첨부파일 | 내부 포털 게시글 변조를 통해 사용자에게 노출된 파일 |
| `poster.jpg` | 위장 파일 | Base64 데이터 추출을 통해 실행 파일 복구에 사용 |
| `recovered_svc.exe` | 악성 실행 파일 | `poster.jpg` 내부 데이터에서 복구 후 실행 |
| `downloader.exe` | 후속 페이로드 | 공격자 서버에서 다운로드되어 후속 파일 생성 수행 |
| `C:\ProgramData\DFIR-Lab\downloader.exe` | 후속 페이로드 경로 | 확산 대상 PC에서 PowerShell에 의해 생성된 파일 |
| `C:\ProgramData\hctf_bettleman\svchost.exe` | 랜섬웨어 실행 파일 | 정상 Windows 경로가 아닌 위치에서 생성 및 로드 |
| `README-HCTF-NOTICE.txt` | 랜섬노트 | 감염 PC 및 확산 대상 PC에서 생성된 랜섬노트 |
| `simulate_infection_target.ps1` | 감염 관련 스크립트 | `-ExecutionPolicy Bypass` 옵션과 함께 실행된 스크립트 |

## 4.4 파일 해시 지표 (SHA256)

> 원본 보고서에는 전체 해시값이 기재되어 있습니다. 여기서는 앞자리만 발췌했으니, 실제 탐지 룰에 쓸 때는 [03-incident-report](../03-incident-report/) 원본 PDF의 전체 해시값을 사용하세요.

| 파일명 | 판단 | SHA256 (앞부분) |
|---|---|---|
| `poster.zip` | 악성 첨부파일 | `ec1b64a72b9d0708a4f9db7b6387a5770e6217b8cc5...` |
| `poster.jpg` | 위장 파일 | `a48d62559680abedcf3e6a477920fddc5f59cf1ffaa...` |
| `2026_교육_팜플렛.pdf.lnk` | 실행 유도 파일 | `777e3e3d30ceee3d2b8e769ba1508462e03efd078d8...` |
| `downloader.exe` | 후속 페이로드 | `5e39374ae21ec0449a3f915c1f387409efbb1168a4f...` |
| `svchost.exe` (위장) | 랜섬웨어 실행 파일 | `5dfdf5bcba58a4f3134dce127f5e8c438b4dd233c05...` |

## 4.5 프로세스 및 명령 실행 지표

| 지표 | 의미 | 관련 로그 |
|---|---|---|
| `x-action-redirect` 내 Base64 페이로드 | React2Shell/RCE 페이로드 확인 | Coraza audit 로그 |
| `RCE_OK` | 원격 명령 실행 성공 문자열 | Coraza audit 로그 디코딩 결과 |
| `uid=1000(webadmin)` | 웹 서버 계정 권한의 명령 실행 확인 | Coraza audit 로그 디코딩 결과 |
| `DATABASE_URL` 조회 | DB 접속 정보 조회 정황 | Linux auditd 로그 |
| `grep` 명령 실행 | 서버 내부 환경 정보 검색 행위 | Linux auditd 로그 |
| `PowerShell Event ID 4104` | 스크립트 블록 실행 및 악성 명령 확인 | Windows PowerShell 로그 |
| `wsmprovhost.exe -Embedding` | WinRM 원격 명령 처리 프로세스 | Sysmon Event ID 1 |
| `recovered_svc.exe → 10.10.30.5:5985` | 최초 감염 PC에서 확산 대상 PC로 WinRM 연결 | Sysmon Event ID 3 |
| `-ExecutionPolicy Bypass` | PowerShell 실행 정책 우회 | Sysmon Event ID 1 |

## 4.6 계정 및 민감정보 지표

| 계정/식별자 | 의미 | 보안상 시사점 |
|---|---|---|
| `webadmin` | RCE 결과로 확인된 웹 서버 실행 계정 | 웹 서비스 계정 권한 최소화 필요 |
| `internal_app` | MongoDB 인증 성공 계정 | 애플리케이션 계정 권한 및 접속 위치 제한 필요 |
| `DATABASE_URL` | DB 접속 문자열 | 환경 변수 및 설정 파일 내 민감정보 보호 필요 |

## 4.7 주요 탐지 로그 이벤트

| 로그/이벤트 | 확인 내용 | 활용 방안 |
|---|---|---|
| Sysmon Event ID 1 | 프로세스 생성, PowerShell 실행, `wsmprovhost.exe` 실행 | 악성 프로세스 및 원격 실행 탐지 |
| Sysmon Event ID 3 | 네트워크 연결, WinRM 5985 연결 | 내부 확산 탐지 |
| Sysmon Event ID 7 | 악성 `svchost.exe` 이미지 로드 | 악성 실행 파일 로드 탐지 |
| Sysmon Event ID 11 | 악성 파일 및 랜섬노트 생성 | 파일 생성 기반 랜섬웨어 탐지 |
| PowerShell Event ID 4104 | 스크립트 블록 실행, 파일 복구 및 다운로드 명령 | PowerShell 기반 악성 행위 탐지 |
