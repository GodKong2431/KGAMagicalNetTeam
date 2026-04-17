# MagicStreets

MagicStreets는 최대 4인 실시간 멀티플레이 액션/잠입 구조를 목표로 한 Unity 프로젝트입니다. 플레이어는 민간인 상태로 잠입하다가 마법사로 변신해 전투를 수행하고, 로비-룸-라운드-상점-보스 레이드로 이어지는 세션을 진행합니다. 현재 프로젝트는 Firebase 로그인, Photon PUN2/Voice 기반 멀티플레이, Photon Custom Property 기반 상태 동기화, 상태 머신 기반 플레이어와 AI, ScriptableObject 기반 마법 및 인벤토리, Timeline + Cinemachine 상호작용 연출, 오브젝트 풀과 NVBlast 파괴 시스템을 중심으로 구성되어 있습니다.

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
- PlayerPrefs / Object Pool / Timeline / NVBlast Fracture

## 핵심 기술 소개

### 1. 실시간 멀티플레이 접속과 세션 흐름

- `TitleManager.cs` : Photon 서버 접속과 로비 진입을 담당하고, 테스트 모드 여부에 따라 바로 룸에 참가하거나 로그인 씬으로 이동합니다.
- `FirebaseAuthManager.cs` : Firebase 의존성 체크, 이메일 회원가입/로그인, 닉네임 갱신, Photon 닉네임 연동을 담당합니다.
- `LobbyManager.cs` : 룸 생성, 참가, 랜덤 매칭, 룸 목록 갱신, 닉네임 변경, 로비 재진입 흐름을 관리합니다.
- `RoomManager.cs` : Host/Ready 상태, 비공개 방, Friendly Fire, 라운드 수, 팀 자금 초기화와 시작 버튼 제어를 포함한 대기실 로직을 담당합니다.
- `GameManager.cs` : 네트워크 플레이어 스폰, 씬 동기화, 팀 자금, 상점 왕복, 라운드 클리어, 승패 판정, 커스텀 프로퍼티 초기화를 함께 관리합니다.
- `RoundManager.cs` : 라운드별 스폰 위치, 필요 자금, 라운드 BGM과 전투 BGM 전환을 관리해 세션 진행 기준을 제공합니다.

### 2. 네트워크 상태 동기화와 권한 모델

- `NetworkProperties.cs` : Player/Room Custom Property 접근과 갱신을 확장 메서드로 통일해 Ready, Alive, Wizard, Money, Round, Friendly Fire 상태를 일관되게 다룹니다.
- `PhotonManager.cs` : 공용 RPC 호출, Photon Custom Property 보조, 관측 값 직렬화 지원 기능을 제공합니다.
- `GameManager.cs`, `RoomManager.cs` : 룸 전역 상태를 Custom Property로 갱신하고 씬 이동 시점에 동기화합니다.
- `BaseAI.cs` : AI 계산은 MasterClient 중심으로 수행하고, 나머지 클라이언트는 상태와 결과를 동기화받는 구조를 사용합니다.

### 3. 플레이어 상태 머신, 입력, 카메라, 미니맵

- `PlayableCharacter.cs` : 플레이어 체력, 이동, 점프, 회피, 상호작용, 카메라, 미니맵 아이콘, 상태 머신과 이벤트를 연결하는 플레이어 중심 엔트리입니다.
- `StateMachine.cs`, `PlayerMoveState.cs`, `PlayerJumpState.cs`, `PlayerDodgeState.cs`, `PlayerAttackState.cs` : 플레이어 행동 전이를 상태 단위로 분리해 이동과 액션 흐름을 구성합니다.
- `PlayerInputHandler.cs` : Input System 액션을 이동, 점프, 공격, 변신, 인벤토리 선택, 상호작용, UI 입력 차단 흐름으로 분리해 전달합니다.
- `PlayerTransformationController.cs` : 민간인에서 마법사로 변신하는 연출과 입력 차단, 모델 전환, Photon 동기화를 담당합니다.
- `PlayerController.cs` : HP UI, 마법 UI, Photon Voice 상태 표시, Friendly Fire 체크, 마법 피격 반응을 처리합니다.
- `ThirdPersonCamera.cs` : Cinemachine 입력 축과 감도, 축 반전, 커서 잠금, UI 상태 연동을 통해 인게임 카메라를 제어합니다.
- `MinimapCamera.cs`, `MapIcon.cs` : 로컬 플레이어 추적형 미니맵 카메라와 오브젝트 아이콘 표시를 담당합니다.

### 4. ScriptableObject 기반 마법과 인벤토리

- `PlayerMagicSystem.cs` : 좌우 손 슬롯 장착, 조준 지점 계산, 마법/아이템 액션 실행, 쿨다운 이벤트 발행을 담당합니다.
- `PlayerInventory.cs` : 최대 슬롯 수, 아이템 수량, 액션 인스턴스 캐싱, 쿨다운 Tick, 아이템 추가/삭제와 UI 갱신을 담당합니다.
- `InventoryWheelLogic.cs` : Q/E 홀드 기반 인벤토리 휠 UI를 열고, 선택한 아이템을 좌우 손 슬롯에 장착하는 흐름을 관리합니다.
- `InventoryDataSO.cs`, `ActionItemDataSO.cs`, `MagicDataSO.cs` : 아이템 메타데이터, 액션 공통 정보, 마법별 데미지/넉백/반경/프리팹 데이터를 분리합니다.
- `ActionBase.cs`, `MagicAction.cs`, `ItemAction.cs` : 데이터와 런타임 행동을 연결하는 공용 액션 추상화를 제공합니다.
- `Fireball`, `LightningStrike`, `Tornado`, `BlackHole`, `Polymorph` 계열 클래스 : 각 마법을 개별 액션으로 구현해 ScriptableObject 데이터와 런타임 행동을 연결합니다.

### 5. AI 경계 시스템, 디버프, 보스 레이드

- `BaseAI.cs` : NavMeshAgent, 상태 머신, 네트워크 상태 동기화, 피격/사망, 디버프, 래그돌 전환을 공통 기반으로 제공합니다.
- `AIStateBase.cs`, `GuardPatrolState.cs`, `GuardChaseState.cs`, `GuardAttackState.cs` 계열 : 경비 AI의 순찰, 추적, 공격, 복귀 흐름을 상태 단위로 나눠 관리합니다.
- `GuardAI.cs`, `GarrisonGuardAI.cs`, `CitizenAI.cs` : 순찰, 추적, 공격, 경계, 귀환, 암살 반응 등 역할별 AI 흐름을 구현합니다.
- `GuardManager.cs` : 경비 스폰 웨이브, 마법 소음 공유, 플레이어 위치 공유, 타이머, 웨이브 이벤트를 관리합니다.
- `DebuffDefinitions.cs` : 기절, 변이, 감속, 처형, 화상 등 상태 이상 정보를 구조체와 인터페이스로 공통화합니다.
- `DragonAI.cs` : 수면, 추적, 전투, 페이즈 전환, 비행 공격, 사망 상태를 포함한 보스 상태 머신을 담당합니다.
- `BossRaidManager.cs` : 플레이어 집결, 보스 기상, 레이드 종료 후 보상 위치 텔레포트를 동기화합니다.

### 6. 상호작용 연출과 컷신 카메라

- `InteractionManager.cs` : 상호작용 요청을 받아 타입별 시스템을 준비하고, Timeline 트랙 바인딩과 위치 보정을 수행합니다.
- `BaseInteractSystem.cs`, `AssassinateInteract.cs` : 플레이어와 AI 간 암살 상호작용 같은 연출형 액션을 공통 인터페이스로 처리합니다.
- `PlayerAssassinateState.cs`, `AIAssassinateState.cs` 계열 : 상호작용을 플레이어/AI 상태 머신과 연결해 실행 중 이동과 입력을 제어합니다.
- `CinemachineController.cs` : 인게임 카메라와 컷신 카메라의 우선순위를 전환해 상호작용 연출을 안정적으로 재생합니다.
- `ProjectManager.cs` : 씬 전역에서 Cinemachine 제어점을 공유하는 진입점입니다.

### 7. UI, 로컬 설정, 로컬라이제이션

- `UIManager.cs` : 로그인 UI 연결, 옵션 패널, 플레이어 정보 패널, 오디오/그래픽/언어 설정, ESC 기반 UI 열기/닫기를 관리합니다.
- `PlayerPrefsDataManager.cs` : 로그인 ID, 언어, BGM/SFX/보이스/마이크 볼륨, 해상도, 마우스 감도와 반전 설정을 로컬에 저장합니다.
- `LocalizationManager.cs`, `DropdownLocalized.cs`, `UI_Settings.cs` : 언어 전환과 로컬라이즈드 문자열/폰트/머티리얼 반영을 담당합니다.

### 8. 채팅, 음성, 사용자 데이터

- `ChattingManager.cs` : Photon RPC 기반 채팅 패널, 엔터 입력, 입장/퇴장 메시지 표시를 처리합니다.
- `PlayerSoundHandler.cs` : Photon Voice Recorder, 마이크 증폭, 타 플레이어 음성 볼륨과 음소거를 UI 설정과 연결합니다.
- `VoiceManager.cs` : 음성 관련 싱글턴 진입점을 제공합니다.
- `FirebaseDBMgr.cs` : 사용자별 색상 선택과 일부 예시 인벤토리 데이터를 Firebase Realtime Database에 저장/로드합니다.

### 9. 사운드, 런타임 최적화, 파괴 시스템, 에디터 자동화

- `SoundManager.cs` : BGM/SFX 재생, 오디오 믹서 볼륨 제어, 사운드 딕셔너리 초기화, 씬 전환 시 BGM 정리를 담당합니다.
- `ObjectPoolManager.cs`, `SinglePoolManager.cs`, `SoundPool.cs` : 반복 생성되는 오디오 소스 같은 리소스를 풀링해 재사용합니다.
- `CoroutineManager.cs` : `WaitForSeconds`, `WaitForSecondsRealtime`, `WaitForFixedUpdate`를 캐싱해 반복 코루틴에서 GC 부담을 줄입니다.
- `FryingPanLogic.cs` : 반복 발사되는 투사체를 내부 큐 기반으로 재사용해 런타임 생성 비용을 줄입니다.
- `BaseSceneChanger.cs` : 씬 진입/이탈 시점의 공통 BGM 처리와 로딩 진입점을 위한 베이스 클래스를 제공합니다.
- `BuildAutomator.cs` : 현재 Build Settings에 등록된 씬을 기준으로 Windows 빌드를 자동 생성합니다.
- `Fracture.cs`, `ChunkGraphManager.cs` : NVBlast 기반 파괴 가능한 메시를 청크 단위로 생성하고 연결 정보를 관리합니다.
- `FractureBaker.cs` : 에디터 윈도우에서 파괴 프리팹을 굽고 메시 에셋을 저장해 파괴 오브젝트 제작 파이프라인을 지원합니다.

## 기술 결정 기록

개별 마법 수치나 보스 패턴 같은 콘텐츠 세부값은 기능 설명 성격이 강하다고 판단해 기술 결정 기록에서는 제외했습니다. 아래 항목은 현재 프로젝트의 핵심 구조와 중요한 지원 기술을 왜 채택했는지에 대한 기록입니다.

### ADR-001. Photon 룸 기반 멀티플레이 채택

- 배경 : 로비에서 룸을 만들고 바로 게임 씬으로 진입하는 최대 4인 실시간 플레이 구조가 필요했습니다.
- 결정 : Photon PUN2 기반의 룸 생성, 준비 상태, 커스텀 프로퍼티, 씬 동기화 구조를 채택했습니다.
- 채택 이유 : 룸 단위 매칭과 씬 전환을 빠르게 구성할 수 있고, 플레이어/룸 속성을 커스텀 프로퍼티로 관리하기 쉬워 현재 프로젝트 흐름에 적합했습니다.

### ADR-002. Firebase 인증과 프로필 데이터 연동

- 배경 : 테스트용 닉네임보다 사용자 계정과 표시 이름을 유지할 수 있는 로그인 구조가 필요했습니다.
- 결정 : Firebase Auth를 로그인 진입점으로 사용하고, 일부 사용자 데이터는 Realtime Database로 연동했습니다.
- 채택 이유 : 이메일 기반 인증을 빠르게 구축할 수 있고, Photon 닉네임과 자연스럽게 연결할 수 있어 멀티플레이 사용자 식별에 유리했습니다.

### ADR-003. Photon Custom Property 기반 상태 동기화

- 배경 : 룸 준비 상태, 생존 여부, 변신 상태, 팀 자금, 라운드 정보 같은 값이 여러 씬과 시스템에 걸쳐 공유되어야 했습니다.
- 결정 : Photon Player/Room Custom Property를 공통 상태 저장소로 사용하고, `NetworkProperties.cs` 확장 메서드로 접근 방식을 통일했습니다.
- 채택 이유 : 별도 동기화 구조를 중복 구현하지 않고도 룸 전역 상태와 플레이어 상태를 일관되게 읽고 갱신할 수 있어 멀티플레이 흐름 관리에 적합했습니다.

### ADR-004. 상태 머신 기반 플레이어와 AI 구조

- 배경 : 플레이어는 이동, 점프, 회피, 공격, 변신, 상호작용을 오가고, AI는 순찰, 추적, 공격, 디버프, 사망 상태를 반복적으로 전환해야 했습니다.
- 결정 : 플레이어와 AI 모두 상태 머신을 중심으로 행동을 분리하는 구조를 채택했습니다.
- 채택 이유 : 입력 처리와 행동 전이를 명확하게 분리할 수 있고, 기능 추가 시 기존 로직을 덜 건드리면서 상태를 확장하기 쉽다고 판단했습니다.

### ADR-005. ScriptableObject 기반 마법 및 인벤토리 데이터 구조

- 배경 : 마법 종류와 아이템이 늘어날수록 공통 속성과 런타임 동작을 분리할 필요가 있었습니다.
- 결정 : `InventoryDataSO` - `ActionItemDataSO` - `MagicDataSO` 계층과 런타임 액션 인스턴스를 결합한 구조를 채택했습니다.
- 채택 이유 : 데이터 수정은 에셋 단위로 관리하고, 런타임 행동은 클래스 단위로 분리할 수 있어 유지보수와 확장에 유리했습니다.

### ADR-006. MasterClient 중심 AI 시뮬레이션 채택

- 배경 : AI 이동과 판정을 모든 클라이언트가 각각 계산하면 동기화 오차와 관리 비용이 커질 수 있었습니다.
- 결정 : AI 상태 계산은 MasterClient 중심으로 수행하고, 나머지 클라이언트는 상태와 결과를 동기화받는 구조를 채택했습니다.
- 채택 이유 : NavMeshAgent, 타이머, 웨이브 스폰, 타깃 공유를 한 곳에서 계산하면 일관성이 높고 네트워크 행동 차이를 줄이기 쉬웠습니다.

### ADR-007. Timeline + Cinemachine 상호작용 연출 프레임워크

- 배경 : 암살 같은 상호작용은 단순 입력 처리만으로는 연출과 카메라 제어를 일관되게 맞추기 어려웠습니다.
- 결정 : Timeline 트랙 바인딩과 Cinemachine 카메라 전환을 공통 InteractionManager로 묶는 구조를 채택했습니다.
- 채택 이유 : 연출형 상호작용을 데이터와 시퀀스 중심으로 관리할 수 있고, 실행자/피실행자/카메라 위치를 한 흐름 안에서 제어하기 좋았습니다.

### ADR-008. PlayerPrefs와 Localization 기반 클라이언트 설정 관리

- 배경 : 해상도, 언어, 사운드, 마우스 감도 같은 설정은 계정과 무관하게 클라이언트별로 즉시 저장되고 다시 불러와져야 했습니다.
- 결정 : 옵션은 PlayerPrefs에 저장하고, 언어 전환은 Localization 패키지와 드롭다운 UI를 결합하는 구조를 채택했습니다.
- 채택 이유 : 구현 비용이 낮고 즉시 반영이 쉬우며, 플레이어 개인 설정을 가볍게 유지하는 데 적합했습니다.

### ADR-009. Photon Voice와 RPC 채팅 기반 커뮤니케이션 구조

- 배경 : 협동 플레이에서는 텍스트 채팅과 실시간 음성 커뮤니케이션이 모두 필요했습니다.
- 결정 : 음성은 Photon Voice, 텍스트 채팅은 Photon RPC 기반 메시지 브로드캐스트로 구성했습니다.
- 채택 이유 : 네트워크 계층을 Photon 생태계 안에서 통일하면 룸 단위 커뮤니케이션을 자연스럽게 묶을 수 있고, UI 설정과도 연결하기 쉬웠습니다.

### ADR-010. 오브젝트 풀링과 코루틴 캐싱 기반 반복 리소스 최적화

- 배경 : 효과음, 반복 투사체, 주기성 코루틴 호출이 누적되면 런타임 중 GC와 생성/파괴 비용이 커질 수 있었습니다.
- 결정 : 반복 자원은 오브젝트 풀과 내부 큐 재사용 구조로 관리하고, 자주 쓰는 코루틴 대기 객체는 캐싱하는 방식으로 처리했습니다.
- 채택 이유 : 런타임 중 GC와 Instantiate/Destroy 비용을 줄일 수 있어 전투와 UI 업데이트가 반복되는 프로젝트에 적합했습니다.

### ADR-011. NVBlast 기반 파괴 오브젝트 제작 파이프라인 도입

- 배경 : 파괴 가능한 구조물을 단순 비주얼 효과가 아니라 실제 청크 단위 물리 오브젝트로 다루고 싶었습니다.
- 결정 : 런타임 파괴 로직은 `Fracture.cs`, `ChunkGraphManager.cs`로 구성하고, 제작 단계에서는 `FractureBaker.cs` 에디터 도구로 프리팹을 베이킹하는 방식을 채택했습니다.
- 채택 이유 : 제작 파이프라인과 런타임 처리 경계를 분리하면 파괴 연출을 반복적으로 조정하기 쉽고, 실제 씬에서는 준비된 파괴 프리팹을 안정적으로 사용할 수 있습니다.

## 핵심 흐름

- 접속과 인증 : `TitleManager.cs`가 Photon `kr` 리전에 접속하고, `LoginManager.cs`와 `FirebaseAuthManager.cs`가 로그인 및 닉네임 준비를 처리합니다.
- 로비와 룸 준비 : `LobbyManager.cs`가 룸 생성과 참가를 담당하고, `RoomManager.cs`가 Ready 상태와 방 옵션을 맞춘 뒤 게임 시작을 동기화합니다.
- 인게임 진입 : `GameManager.cs`가 네트워크 플레이어를 스폰하고, `RoundManager.cs`가 스폰 위치와 라운드 기준을 제공하며, `PlayableCharacter.cs`가 입력, 상태 머신, 변신, 마법, 카메라를 연결합니다.
- 전투와 잠입 : `GuardManager.cs`와 AI 상태 머신이 순찰 및 추적을 수행하고, 플레이어는 `PlayerMagicSystem.cs`와 `PlayerInventory.cs`를 통해 마법과 아이템을 사용하며, 필요 시 `InteractionManager.cs`가 암살 연출을 실행합니다.
- 보스 레이드 : 일반 라운드 흐름 이후 `BossRaidManager.cs`와 `DragonAI.cs`가 집결, 전투, 보상 위치 이동까지 이어지는 보스 흐름을 처리합니다.
- UI와 커뮤니케이션 : `UIManager.cs`가 옵션과 HUD를 관리하고, `ChattingManager.cs`와 `PlayerSoundHandler.cs`가 텍스트 채팅과 음성 커뮤니케이션을 연결합니다.
- 저장과 지원 시스템 : `PlayerPrefsDataManager.cs`, `FirebaseDBMgr.cs`, `SoundManager.cs`, `CoroutineManager.cs`, `FractureBaker.cs`가 설정 저장, 사용자 데이터, 사운드, 최적화, 파괴 프리팹 제작을 보조합니다.
