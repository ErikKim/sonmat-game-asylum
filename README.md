# 배트맨: 아캄 어사일럼 — 손맛 햅틱 게임 프로필

- id `asylum`, 실행 파일 `ShippingPC-BmGame.exe`(32비트 언리얼 3), 게임 연결 `hook.json`(훅 DLL `dshaptics_ue3_x86.dll`, 부품 `sonmat-hook-ue3`)
- `haptics.json` = 느낌(규칙·파형·트리거·클립 지시·소리 위치), `hook.json` = 게임 분석 결과(훅 DLL·바이트 패턴). 다른 파일은 거기서 이름으로 부르는 표다
- 소리 파일은 없다: 앱이 사용자 PC 에서 그 사람의 게임 파일로 만든다(패키지의 FMOD 프로젝트, 부품 `sonmat-lzo1x`)
- 앱은 이 레포를 `<games>/asylum/` 에 받는다(폴더 이름 = id)
- 게임은 XInput 으로 패드를 읽는다: 스팀 입력을 켜고, 게임 진동은 끈다
