# evidence

Wazuh, Coraza WAF, MongoDB, Sysmon, PowerShell 로그의 원본 캡처를 시간순으로 모아두는 폴더입니다.

이번 정리에서는 원본 보고서 PDF(`docs/03-incident-report/`) 안에 로그 스크린샷이 이미 단계별로 잘 정리되어 있어서 별도로 이미지를 다시 뽑아 중복해 넣지는 않았습니다. 만약 로그 원본(json/txt export 등)을 따로 보관하고 있다면, 아래처럼 단계별 하위 폴더로 나눠 넣는 걸 추천합니다.

```
evidence/
├── 01-initial-access/     # DMZ Web 최초 접근, RCE 실행 로그
├── 02-db-tamper/          # DATABASE_URL 조회, MongoDB 접속/변조 로그
├── 03-user-infection/     # poster.zip 다운로드~실행, Sysmon 로그
├── 04-lateral-movement/   # WinRM 기반 내부 확산 로그
└── 05-ransomware/         # 랜섬노트 생성, 피해 확인 로그
```

민감정보(DB 비밀번호, JWT_SECRET 등)가 로그에 그대로 남아있다면 커밋 전에 반드시 마스킹해주세요.
