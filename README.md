 MagicStreets

MagicStreets는 최대 4인 실시간 멀티플레이 액션/잠입 구조를 목표로 한 Unity 프로젝트입니다. 플레이어는 민간인 상태로 잠입하다가 마법사로 변신해 전투를 수행하고, 로비-룸-라운드-상점-보스 레이드로 이어지는 세션을 진행합니다. 현재 프로젝트는 Firebase 로그인, Photon PUN2/Voice 기반 멀티플레이, 상태 머신 기반 플레이어와 AI, ScriptableObject 기반 마법 및 인벤토리, Timeline + Cinemachine 상호작용 연출을 중심으로 구성되어 있습니다.

## 기술 스택

- Unity 6000.2.10f1
- URP 17.2.0
- Photon PUN2 2.51
- Photon Voice 2
- Firebase Auth / Realtime Database 13.6.0
- Input System 1.14.2
- AI Navigation 2.0.9
- Cinemachine 3.1.5
- Localization 1.5.9
- PlayerPrefs / Object Pool / Timeline Interaction / NVBlast Fracture

## 핵심 기술 소개

### 1. 멀티플레이 접속과 세션 흐름

- `TitleManager.cs` : Photon 서버 접속과 로비 진입을 담당하고, 테스트 모드 여부에 따라 바로 룸에 참가하거나 로그인 씬으로 이동합니다.
- `FirebaseAuthManager.cs` : Firebase 의존성 체크, 이메일 회원가입/로그인, 닉네임 갱신, Photon 닉네임 연동을 담당합니다.
- `LobbyManager.cs` : 룸 생성, 참가, 랜덤 매칭, 룸 목록 갱신, 닉네임 변경, 로비 재진입 흐름을 관리합니다.
- `RoomManager.cs` : Host/Ready 상태, 비공개 방, Friendly Fire, 라운드 수, 팀 자금 초기화와 시작 버튼 제어를 포함한 대기실 로직을 담당합니다.
- `GameManager.cs` : 네트워크 플레이어 스폰, 씬 동기화, 팀 자금, 상점 왕복, 라운드 클리어, 승패 판정, 커스텀 프로퍼티 초기화를 함께 관리합니다.
- `PhotonManager.cs` : 공용 RPC 호출, Photon Custom Property 보조, 관측 값 직렬화 지원 기능을 제공합니다.

### 2. 플레이어 제어와 변신형 액션 구조

- `PlayableCharacter.cs` : 플레이어 체력, 이동, 점프, 회피, 상호작용, 카메라, 미니맵 아이콘, 상태 머신과 이벤트를 연결하는 플레이어 중심 엔트리입니다.
- `PlayerInputHandler.cs` : Input System 액션을 이동, 점프, 공격, 변신, 인벤토리 선택, 상호작용, UI 입력 차단 흐름으로 분리해 전달합니다.
- `PlayerTransformationController.cs` : 민간인에서 마법사로 변신하는 연출과 입력 차단, 모델 전환, Photon 동기화를 담당합니다.
- `PlayerMagicSystem.cs` : 좌우 손 슬롯 장착, 조준 지점 계산, 마법/아이템 액션 실행, 쿨다운 이벤트 발행을 담당합니다.
- `PlayerController.cs` : HP UI, 마법 UI, Photon Voice 상태 표시, Friendly Fire 체크, 마법 피격 반응을 처리합니다.

### 3. ScriptableObject 기반 인벤토리와 액션 데이터

- `PlayerInventory.cs` : 최대 슬롯 수, 아이템 수량, 액션 인스턴스 캐싱, 쿨다운 Tick, 아이템 추가/삭제와 UI 갱신을 담당합니다.
- `InventoryWheelLogic.cs` : Q/E 홀드 기반 인벤토리 휠 UI를 열고, 선택한 아이템을 좌우 손 슬롯에 장착하는 흐름을 관리합니다.
- `InventoryDataSO.cs`, `ActionItemDataSO.cs`, `MagicDataSO.cs` : 아이템 메타데이터, 액션 공통 정보, 마법별 데미지/넉백/반경/프리팹 데이터를 분리합니다.
- `Fireball`, `LightningStrike`, `Tornado`, `BlackHole`, `Polymorph` 계열 클래스 : 각 마법을 개별 액션으로 구현해 ScriptableObject 데이터와 런타임 행동을 연결합니다.
- `DebuffDefinitions.cs` : 기절, 변이, 감속, 처형, 화상 등 상태 이상 정보를 구조체와 인터페이스로 공통화합니다.

### 4. 잠입 AI와 보스 레이드 구조

- `BaseAI.cs` : NavMeshAgent, 상태 머신, 네트워크 상태 동기화, 피격/사망, 디버프, 래그돌 전환을 공통 기반으로 제공합니다.
- `GuardAI.cs`, `GarrisonGuardAI.cs`, `CitizenAI.cs` : 순찰, 추적, 공격, 경계, 귀환, 암살 반응 등 역할별 AI 흐름을 구현합니다.
- `GuardManager.cs` : 경비 스폰 웨이브, 마법 소음 공유, 플레이어 위치 공유, 타이머, 웨이브 이벤트를 관리합니다.
- `DragonAI.cs` : 수면, 추적, 전투, 페이즈 전환, 비행 공격, 사망 상태를 포함한 보스 상태 머신을 담당합니다.
- `BossRaidManager.cs` : 플레이어 집결, 보스 기상, 레이드 종료 후 보상 위치 텔레포트를 동기화합니다.

### 5. 상호작용 연출과 카메라 프레임워크

- `InteractionManager.cs` : 상호작용 요청을 받아 타입별 시스템을 준비하고, Timeline 트랙 바인딩과 위치 보정을 수행합니다.
- `BaseInteractSystem.cs`, `AssassinateInteract.cs` : 플레이어와 AI 간 암살 상호작용 같은 연출형 액션을 공통 인터페이스로 처리합니다.
- `PlayerAssassinateState.cs`, `AIAssassinateState.cs` 계열 : 상호작용을 플레이어/AI 상태 머신과 연결해 실행 중 이동과 입력을 제어합니다.
- `CinemachineController.cs` : 인게임 카메라와 컷신 카메라의 우선순위를 전환해 상호작용 연출을 안정적으로 재생합니다.
- `ProjectManager.cs` : 씬 전역에서 Cinemachine 제어점을 공유하는 얇은 싱글턴 진입점입니다.

### 6. UI, 로컬 설정, 커뮤니케이션 구조

- `UIManager.cs` : 로그인 UI 연결, 옵션 패널, 플레이어 정보 패널, 오디오/그래픽/언어 설정, ESC 기반 UI 열기/닫기를 관리합니다.
- `PlayerPrefsDataManager.cs` : 로그인 ID, 언어, BGM/SFX/보이스/마이크 볼륨, 해상도, 마우스 감도와 반전 설정을 로컬에 저장합니다.
- `LocalizationManager.cs`, `DropdownLocalized.cs`, `UI_Settings.cs` : 언어 전환과 로컬라이즈드 문자열/폰트/머티리얼 반영을 담당합니다.
- `ChattingManager.cs` : Photon RPC 기반 채팅 패널, 엔터 입력, 입장/퇴장 메시지 표시를 처리합니다.
- `PlayerSoundHandler.cs` : Photon Voice Recorder, 마이크 증폭, 타 플레이어 음성 볼륨과 음소거를 UI 설정과 연결합니다.
- `FirebaseDBMgr.cs` : 사용자별 색상 선택과 일부 예시 인벤토리 데이터를 Firebase Realtime Database에 저장/로드합니다.

### 7. 런타임 공용 시스템과 최적화

- `SoundManager.cs` : BGM/SFX 재생, 오디오 믹서 볼륨 제어, 사운드 딕셔너리 초기화, 씬 전환 시 BGM 정리를 담당합니다.
- `ObjectPoolManager.cs`, `SinglePoolManager.cs`, `SoundPool.cs` : 반복 생성되는 오디오 소스 같은 리소스를 풀링해 재사용합니다.
- `FryingPanLogic.cs` : 반복 발사되는 투사체를 내부 큐 기반으로 재사용해 런타임 생성 비용을 줄입니다.
- `BaseSceneChanger.cs` : 씬 진입/이탈 시점의 공통 BGM 처리와 로딩 진입점을 위한 베이스 클래스를 제공합니다.
- `BuildAutomator.cs` : 현재 Build Settings에 등록된 씬을 기준으로 Windows 빌드를 자동 생성합니다.
- `Fracture.cs`, `ChunkGraphManager.cs` : NVBlast 기반 파괴 가능한 메시를 청크 단위로 생성하고 연결 정보를 관리합니다.

## 기술 결정 기록

개별 마법 성능 수치나 보스 패턴 같은 콘텐츠 세부값은 기능 설명 성격이 강하다고 판단해 기술 결정 기록에서는 제외했습니다. 아래 항목은 현재 프로젝트의 핵심 구조를 왜 채택했는지에 대한 기록입니다.

### ADR-001. Photon 룸 기반 멀티플레이 채택

- 배경 : 로비에서 룸을 만들고 바로 게임 씬으로 진입하는 최대 4인 실시간 플레이 구조가 필요했습니다.
- 결정 : Photon PUN2 기반의 룸 생성, 준비 상태, 커스텀 프로퍼티, 씬 동기화 구조를 채택했습니다.
- 채택 이유 : 룸 단위 매칭과 씬 전환을 빠르게 구성할 수 있고, 플레이어/룸 속성을 커스텀 프로퍼티로 관리하기 쉬워 현재 프로젝트 흐름에 적합했습니다.

### ADR-002. Firebase 인증과 프로필 데이터 연동

- 배경 : 테스트용 닉네임보다 사용자 계정과 표시 이름을 유지할 수 있는 로그인 구조가 필요했습니다.
- 결정 : Firebase Auth를 로그인 진입점으로 사용하고, 일부 사용자 데이터는 Realtime Database로 연동했습니다.
- 채택 이유 : 이메일 기반 인증을 빠르게 구축할 수 있고, Photon 닉네임과 자연스럽게 연결할 수 있어 멀티플레이 사용자 식별에 유리했습니다.

### ADR-003. 상태 머신 기반 플레이어와 AI 구조

- 배경 : 플레이어는 이동, 점프, 회피, 공격, 변신, 상호작용을 오가고, AI는 순찰, 추적, 공격, 디버프, 사망 상태를 반복적으로 전환해야 했습니다.
- 결정 : 플레이어와 AI 모두 상태 머신을 중심으로 행동을 분리하는 구조를 채택했습니다.
- 채택 이유 : 입력 처리와 행동 전이를 명확하게 분리할 수 있고, 기능 추가 시 기존 로직을 덜 건드리면서 상태를 확장하기 쉽다고 판단했습니다.

### ADR-004. ScriptableObject 기반 마법 및 인벤토리 데이터 구조

- 배경 : 마법 종류와 아이템이 늘어날수록 공통 속성과 런타임 동작을 분리할 필요가 있었습니다.
- 결정 : `InventoryDataSO` - `ActionItemDataSO` - `MagicDataSO` 계층과 런타임 액션 인스턴스를 결합한 구조를 채택했습니다.
- 채택 이유 : 데이터 수정은 에셋 단위로 관리하고, 런타임 행동은 클래스 단위로 분리할 수 있어 유지보수와 확장에 유리했습니다.

### ADR-005. Timeline + Cinemachine 상호작용 연출 프레임워크

- 배경 : 암살 같은 상호작용은 단순 입력 처리만으로는 연출과 카메라 제어를 일관되게 맞추기 어려웠습니다.
- 결정 : Timeline 트랙 바인딩과 Cinemachine 카메라 전환을 공통 InteractionManager로 묶는 구조를 채택했습니다.
- 채택 이유 : 연출형 상호작용을 데이터와 시퀀스 중심으로 관리할 수 있고, 실행자/피실행자/카메라 위치를 한 흐름 안에서 제어하기 좋았습니다.

### ADR-006. MasterClient 중심 AI 시뮬레이션 채택

- 배경 : AI 이동과 판정을 모든 클라이언트가 각각 계산하면 동기화 오차와 관리 비용이 커질 수 있었습니다.
- 결정 : AI 상태 계산은 MasterClient 중심으로 수행하고, 나머지 클라이언트는 상태와 결과를 동기화받는 구조를 채택했습니다.
- 채택 이유 : NavMeshAgent, 타이머, 웨이브 스폰, 타깃 공유를 한 곳에서 계산하면 일관성이 높고 네트워크 행동 차이를 줄이기 쉬웠습니다.

### ADR-007. 로컬 설정과 반복 리소스는 경량 저장/재사용 구조로 처리

- 배경 : 옵션 값은 빠르게 저장되어야 했고, 효과음과 반복 투사체는 생성/파괴 비용을 줄일 필요가 있었습니다.
- 결정 : 사용자 설정은 PlayerPrefs에 저장하고, 반복 자원은 오브젝트 풀과 내부 큐 재사용 구조로 관리했습니다.
- 채택 이유 : 구현 부담이 적고 즉시 반영이 쉬우며, 전투 중 반복 생성되는 리소스의 비용을 줄이는 데 적합했습니다.

## 핵심 흐름

- 접속과 인증 : `TitleManager.cs`가 Photon `kr` 리전에 접속하고, `LoginManager.cs`와 `FirebaseAuthManager.cs`가 로그인 및 닉네임 준비를 처리합니다.
- 로비와 룸 준비 : `LobbyManager.cs`가 룸 생성과 참가를 담당하고, `RoomManager.cs`가 Ready 상태와 방 옵션을 맞춘 뒤 게임 시작을 동기화합니다.
- 인게임 진입 : `GameManager.cs`가 네트워크 플레이어를 스폰하고, `PlayableCharacter.cs`가 입력, 상태 머신, 변신, 마법, 상호작용을 연결합니다.
- 잠입과 전투 : `GuardManager.cs`와 AI 상태 머신이 순찰 및 추적을 수행하고, 플레이어는 `PlayerMagicSystem.cs`와 `PlayerInventory.cs`를 통해 마법과 아이템을 사용합니다.
- 라운드 진행 : `GameManager.cs`가 팀 자금, 상점 이동, 라운드 클리어, 승패 판정을 관리하고, 최종적으로 `BossRaidManager.cs`와 `DragonAI.cs`가 보스 레이드 흐름을 이어받습니다.
- UI와 로컬 데이터 : `UIManager.cs`가 옵션과 HUD를 관리하고, `PlayerPrefsDataManager.cs`와 `FirebaseDBMgr.cs`가 로컬 설정 및 일부 사용자 데이터를 저장합니다.
