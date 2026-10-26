# Batman: Arkham Asylum GOTY — 받아 온 것 (2026-10-18)

설치본: PC `C:\Program Files (x86)\Steam\steamapps\common\Batman Arkham Asylum GOTY` (한글 패치·자막 패치·FMOD 버퍼 패치가 들어간 exe, md5 34b15246…). 원본 exe = `C:\arkham_mods\backup\asylum_orig\ShippingPC-BmGame.exe` (md5 265f1dd9…). 두 exe 모두 아래 바이트 줄이 한 곳씩만 맞음.

## 1. 훅 — 만들었고 빌드됨, 게임 안에서 켜 본 적 없음
- 엔진: Unreal Engine 3 (576/21), **32비트**. 소리 = FMOD Ex + FMOD Designer 이벤트, exe 안에 정적 링크(별도 DLL 없음). exe는 포장 안 됨(코드 구역 엔트로피 6.5) → 파일만 읽어 찾음. **확인됨**
- 건 곳: `UAudioComponent::Play`(원본 0x791770) — 소리 재생은 다 여기를 지남. 끝: `UAudioComponent::Cleanup`(0x799ca0, "재생 중이었나" 0x60 & 0x80을 스스로 확인). 이름: `FName::ToString`(0x5d5690). 근거: `AudioComponentexecPlay` 의 네이티브 표, 게임 자체 소리 목록 출력 코드("%d (%s # %s): FMOD [%s.%s]")가 쓰는 칸(+0x40 주인, +0x4c SoundCue, cue+0x8c 프로젝트). **확인됨(정적)**
- 이벤트 줄: `ue:<FMOD 프로젝트>.<SoundCue>|<주인 액터>`, 끝 `ue_end:<같은 이름>|<주인>`(EndFilter 에 든 이름만). **형식은 설계, 실제 줄은 미확인**
- DLL: `dll/dshaptics_ue3.cpp` → `dshaptics_ue3_x86.dll`, 로더 `dshaptics_loader_x86.dll`(version.dll). exe 가 VERSION.dll 을 직접 가져옴, Binaries 에 다른 version.dll 없음(DSOAL 의 dsound.dll 만 있음). **빌드 확인됨(PC, VS x86)**
- 주입은 안 함: 64비트 엔진이 32비트 게임에 LoadLibraryW 주입하면 주소가 틀려 게임이 죽음 → `hapticsd.maybe_inject` 가 32비트 DLL 이면 건너뜀(로더만).

## 2. 이름표
- 출처: 게임 파일 그 자체. SoundCue 이름 = FMOD 이벤트 이름(`BmGame.u` 의 SoundCue 538개로 대조). **확인됨**
- 개수: FMOD 프로젝트 580개, 이벤트 1,945개(`tools/ue3_fmod.py`, 1차 스캔). 그중 이벤트 정의가 다른 형식이라 아직 못 읽는 것 15개(탐정 모드 화면음, 젤 폭발 ExplodeFinal·debris 등).
- 음원 대조: 소리 정의가 가리키는 음원 5,528개 중 5,337개(96.5%)가 FSB 의 그 번호 음원 이름과 일치. 안 맞는 것은 리더가 이름으로 다시 찾음.
- 분류(프로필 칸에 걸린 수): 발소리 34 · 내 타격 17 · 점프 16 · 착지 15 · 가벼운 타격(배터랭 등) 13 · 휘두름 13 · 느껴지는 바닥 13 · 세계(붕괴·종 낙하 등) 12 · 폭발 11 · 맞음 11 · 회피 7 · 도구 꺼냄 6 · 손 짚기 6 · 크게 맞음 5 · 막기 5 · 쏨 4 · 카운터 4 · 마무리 4 · 뼈 부러짐 3 · 쓰러짐 3 · 큰 휘두름 2 · 괴성 2. 조용히 둘 것(음악·UI·테이프·적 목소리 등) 766.

## 3. 나/남 구분
- 프로젝트 접두어가 주인: `PL_` 배트맨, `EN_` 적, `NPC_`, `DV_` 배트맨 도구. 이벤트가 동작: `Batman_Punch_Enemy_Face`(내가 때림), `Enemy_Punch_Batman_Face`(내가 맞음). **확인됨(이름)**
- 훅은 주인 액터 이름도 줌 — 배트맨 액터 이름은 **미확인**(첫 판 기록에서). 지금 `player: "."`.

## 4. 바닥·상태
- 바닥 = 프로젝트: `PL_FootSteps_<바닥>SFX.FootStep`(콘크리트·돌·타일·나무·카펫·고무·금속·철망·사다리·유리·깨진 유리·흙·풀·웅덩이·깊은 물·바퀴벌레 등). 스위치 아님 — 이름에 들어 옴. **확인됨(이름)**
- 활강: `PL_CapeSFX.bm_glider_open` ~ `bm_glider_close`. 갈고리 당겨짐: `DV_GrappleGunSFX.Start/Loop/End`. 프로필 `beds` 로 넣음.

## 5. 음원
- 게임 패키지(.upk/.umap/.u, 2,606개) 안 `RFMODSound` 객체 = FEV1(이벤트·소리 정의) + FSB4(MP3, 48 kHz). 패키지는 UE3 LZO 덩어리 압축 → 우리 `dll/lzo1x.c`(공개 형식대로, liblzo2 와 바이트 일치 확인). 전체 읽기 15초(PC). 키 없음.
- 배트맨 소리 대부분은 `BmGame.u` 안(전투·망토·도구·바닥별 발소리), 장소 소리는 각 맵.
- 디코딩: vgmstream(FSB4 MP3) 확인(맥).

## 6. 게임 자체 진동·입력
- XInput(XINPUT1_3) → Steam Input 켜고 게임 진동 끄기. exe 에 `ForceFeedbackWaveform` 이 있음(원래 진동 순간은 아직 안 읽음 — 분류 근거로만 쓸 것).
- 컷씬: 실사 영상(.bik, 15 GB)은 Bink 가 소리를 직접 내서 훅에 안 옴 → 진동 없음. 엔진 안 컷씬·연출은 `SP_*` 이벤트로 옴.

## 7. 빈도 기록
- 없음. 손맛 켜고 이 게임을 해 본 적 없음.

## 8. 못 한 것
- 훅을 게임 안에서 한 번도 안 돌림(PC 가 18:30쯤 꺼져서 리더도 PC 에서 아직 안 돌림).
- 번호 붙은 컷씬 프로젝트(115_JokerOnCell … 705_BreakingNews)는 맵 쪽이라 새 해석기로 다시 읽어야 함(맥 사본은 BmGame.u 뿐).
- 이벤트의 첫 층만 씀(게임 변수로 켜지는 층 — 막힘·세기 — 은 안 따라감).
- 버튼(10-19 사용자): 갈고리 = R1(어댑티브 트리거 없음 → 트리거 안 검), R2 = 배터랭·다른 도구.
- R2 트리거는 도구별(10-19 사용자 "베트랑은 활처럼, 스프레이는 지속적인 강한 트리거처럼"): 꺼낼 때 소리(`Batarang_Draw`·`BatGel TakeOut`·`LineLauncher_GetOut`)에 걸고 넣을 때 소리(`Putaway`)에 푼다. 배터랭 = 활(듀얼센스 활 효과 0x22, 엔진에 10-19 추가: 당기다 탁 풀림), 젤 = 끝까지 강한 저항, 줄 발사기 = 무거운 딸깍. 소닉 배터랭·배트클로는 꺼낼 때 소리를 못 찾아 비움. 꺼낼 때 소리가 실제로 오는지는 첫 판 기록에서 확인
- L2(도구 조준, 누르고 있음) 트리거는 안 넣음 — 조준 시작 소리가 무엇인지 기록이 있어야 함.
