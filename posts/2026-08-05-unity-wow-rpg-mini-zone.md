# Unity WoW식 RPG 다음 스텝 — 미니 존(배경) 한 조각

> 상태: published  
> URL: https://moyeo.blogspot.com/2026/08/unity-wow-rpg.html  
> 작성일: 2026-08-05  
> 카테고리: 개발 노트  
> 주제: Unity + AI로 WoW식 RPG 2단계 — Terrain·스카이·라이팅·랜드마크로 미니 존  
> 이전 글: [Unity로 WoW를 닮은 RPG — AI와 첫 질문부터 걷기까지](https://moyeo.blogspot.com/2026/08/unity-wow-rpg-ai.html)

지난번에 WASD와 우클릭 시점이 돌아갔다.  
그다음이 점프일까, 애니일까, 전투일까—아니면 **배경**일까.

결론부터 말하면, 지금은 **미니 존(배경 한 조각)**이 맞다.  
흰 Plane 위를 걷는 캡슐은 “조작 데모”이지, 아직 RPG 구간의 느낌이 아니다.  
와우를 닮게 만드는 두 번째 체감은, 캐릭터가 **어디에 서 있는지**다.

다만 “대륙 전체 배경”이 아니다.  
**200~400m 짜리 야외 존 하나**—걸어서 가장자리까지 1~2분—만 만든다.

## 왜 배경이 다음인가

| 후보 | 지금 하기 | 미루는 이유 |
|------|-----------|------------|
| **미니 존(배경)** | ✅ | 이동·카메라가 “장소”를 만나야 WoW 감각이 붙음 |
| 점프 | 나중 | 한 줄이면 되지만, 존이 없으면 체감이 약함 |
| 애니메이션 | 그다음 | 슬라이딩 캡슐 문제는 크지만, 배경 없는 애니도 허전함 |
| 전투·타겟팅 | 더 나중 | 입력·UI·판정이 한꺼번에 커짐 |
| 오픈월드 | ❌ | 지난 글과 같은 함정—범위가 AI를 삼킴 |

순서를 그림으로 보면:

```text
[1] 이동+카메라  ✓ 완료
[2] 미니 존(배경) ← 오늘
[3] 걷기 애니(Idle/Walk/Run)
[4] 점프·카메라 충돌
[5] 타겟팅·기본 공격
```

배경을 “예쁜 맵”이 아니라 **플레이 테스트용 상자**로 생각하면 부담이 줄어든다.

## 전제: 존의 스펙을 네 줄로

AI에게 맵을 시키기 전에, 내가 먼저 적는다.

| 항목 | 예시 (오늘 쓸 값) |
|------|------------------|
| **크기** | 대략 256×256 Terrain (또는 비슷한 Plane 묶음) |
| **분위기** | 초원/얕은 언덕, 낮, 맑음 (엘윈 숲을 “복제”하지 말 것) |
| **랜드마크 3개** | 돌무더기, 쓰러진 나무(긴 큐브), 작은 폐허(박스 3개) |
| **스폰** | 존 중앙 근처 Empty `PlayerSpawn` |

상표·월드 이름을 베끼지 않는다.  
필요한 건 **야외에서 카메라가 멀리 보이는 공간**뿐이다.

```text
Mini Zone (한 조각) — 대륙이 아님
+--------------------------------------------------+
|  [Rock]            (Spawn)           [Log]       |
|                         o                        |
|                              [Ruin]              |
+--------------------------------------------------+
```

## 1) AI에게 던질 첫 질문 (복붙용)

이전 글의 이동·카메라가 **이미 있는 프로젝트**를 전제로 한다.

```text
이어서 진행. 기존 PlayerMover / CameraOrbit / CharacterController는 깨지 마.
목표: WoW식 3인칭 RPG용 "미니 존" 하나. 오픈월드·마을·던전 금지.
환경:
- URP, Unity 6 또는 2022 LTS
- Terrain 256x256 (또는 동등한 야외 바닥) + 얕은 언덕
- Skybox + Directional Light(낮) + 약한 Fog
- 랜드마크 3개: RockCluster, FallenLog, SmallRuin (Primitives로 충분)
- Empty PlayerSpawn (플레이어 시작 위치)
산출물:
1) Hierarchy 이름 규칙
2) 클릭 경로(메뉴)로 Terrain/Lighting/Fog 설정 순서
3) 임시 머티리얼 색만으로도 OK — 고퀄 텍스처 팩 강요 금지
4) Play 모드 체크리스트 5줄
5) 카메라가 지형에 박히면 다음 단계에서 다룰 예정 — 오늘은 존만
끝에 "다음에 넣을 기능 후보 1개"만 (애니메이션 권장).
```

범위가 새면:

```text
대륙·마을·퀘스트 NPC 제외. Terrain+라이팅+랜드마크 3개+Spawn만.
```

<!-- ADSENSE: mid -->

## 2) Unity에서 손으로 확인할 뼈대

AI 설명과 함께, 에디터에서 이 순서를 한 번 밟는다.

| 단계 | 할 일 | 완료 감각 |
|------|------|-----------|
| 1 | 기존 Plane이 있으면 비활성/삭제 | 바닥이 Terrain으로 바뀜 |
| 2 | `GameObject → 3D Object → Terrain` | 넓은 땅이 보임 |
| 3 | Raise/Lower로 **얕은** 언덕만 | 절벽 미로 금지 |
| 4 | Window → Rendering → Lighting: Environment에 Skybox | 하늘이 회색이 아님 |
| 5 | Fog 체크, Density는 아주 약하게 | 지평선이 살짝 흐림 |
| 6 | Directional Light 각도·intensity 조정 | 그림자가 너무 검지 않음 |
| 7 | Primitives로 랜드마크 3개 + 이름 부여 | Hierarchy에서 식별 가능 |
| 8 | `PlayerSpawn`에 Player 위치 맞춤 | Play 시 존 중앙 근처 |

하이어라키 예:

```text
Environment
  ├─ Terrain
  ├─ Directional Light
  ├─ Landmarks
  │    ├─ RockCluster
  │    ├─ FallenLog
  │    └─ SmallRuin
  └─ PlayerSpawn
Player
CameraRig
  └─ …
```

**함정 1:** Terrain Paint에 시간을 다 쓰면 안 된다.  
오늘은 “걸어 다닐 상자”다. 스탬프 아트는 애니 다음이다.

**함정 2:** 에셋 스토어에서 숲 팩을 잔뜩 넣기.  
AI에게도 못 박는다.

```text
유료/대형 에셋 팩 제안 금지. Unity Primitive + 기본 URP Lit 색만.
```

## 3) 라이팅·안개 — “야외” 한 줄 설정

URP에서 자주 쓰는 최소값 감각이다. 숫자는 프로젝트마다 다시 튜닝한다.

| 항목 | 시작값 감각 |
|------|------------|
| Directional Light Intensity | 1.0~1.5 |
| Light Rotation | X 40~55° (낮) |
| Fog Mode | Exponential 또는 Linear |
| Fog Density / End | “멀리만 흐리게” — 발밑이 안 보이면 과함 |
| Ambient | Skybox 기여가 너무 어두우면 Environment Lighting 확인 |

AI에게 수치를 물어보되, **한 번에 프리셋 열 개를 받지 말고** Play 보면서 두세 번만 고친다.

```text
지금 씬이 너무 어둡다. Directional Light와 Fog만 조정 가이드를 짧게.
다른 시스템 추가 금지.
```

## 4) 스폰과 경계 — 테스트가 편해지게

미니 존에서 제일 많이 하는 실수: Play마다 플레이어가 언덕 밖·지하에 있다.

간단한 스폰 정렬용 스크립트(선택):

```csharp
using UnityEngine;

public class SpawnAtPoint : MonoBehaviour
{
    public Transform spawnPoint;

    void Start()
    {
        if (spawnPoint == null) return;
        var cc = GetComponent<CharacterController>();
        if (cc != null) cc.enabled = false;
        transform.position = spawnPoint.position;
        transform.rotation = spawnPoint.rotation;
        if (cc != null) cc.enabled = true;
    }
}
```

| 필드 | 연결 |
|------|------|
| `SpawnAtPoint.spawnPoint` | `PlayerSpawn` |
| Player 위치 | Spawn과 같은 높이(+0.1 정도) |

경계는 당장 Invisible Wall까지 갈 필요 없다.  
대신 Terrain 가장자리에 **색이 다른 큐브 몇 개**만 세워 “여기부터 밖”을 눈에 보이게 해도 충분하다.

## 5) Play 모드 체크리스트

| # | 확인 |
|---|------|
| 1 | WASD·우클릭이 **이전과 같이** 동작하는가 (회귀 없음) |
| 2 | 하늘·지평선이 Plane 시절과 확실히 다른가 |
| 3 | 랜드마크 3개를 걸어서 모두 찾아갈 수 있는가 |
| 4 | 스폰에서 시작해 낙하/끼임이 없는가 |
| 5 | 카메라가 가끔 땅에 박혀도, 오늘은 존 완성으로 끝내는가 |

막히면:

```text
미니 존 작업만. PlayerMover/CameraOrbit 로직 변경 금지.
증상: (예: Terrain 위에 서면 천천히 가라앉음 / 스폰이 지하)
Hierarchy와 관련 컴포넌트 설정을 짧게 진단해줘.
```

## 6) 다음에 올 것 (예고)

미니 존이 끝나면 추천 순서는 이렇다.

| 다음 | 이유 |
|------|------|
| **Idle/Walk/Run 애니** | 배경이 생긴 뒤 “캐릭터다움”이 가장 크게 오른다 |
| 카메라-지형 충돌 | 언덕이 있어야 의미가 있다 |
| 점프 | 지형 기복이 있을 때 재미가 생긴다 |
| 타겟팅·공격 | 존과 이동이 안정된 뒤 |

오늘 AI 답변 끝에 나올 “다음 후보 1개”는 **애니메이션**이면 합격이다.

## 손에 남길 최소 세트

| # | 완료 조건 |
|---|-----------|
| 1 | 범위를 “미니 존”으로 못 박고 대륙을 거절했다 |
| 2 | Terrain(또는 동급 야외 바닥) + Skybox + Light + Fog |
| 3 | 랜드마크 3개 + `PlayerSpawn` |
| 4 | 기존 이동·카메라가 깨지지 않았다 |
| 5 | 걸어서 존을 한 바퀴 돌 수 있다 |

배경 만들기가 “해야 하냐”면—**해야 한다. 다만 작게.**  
와우를 닮은 RPG의 두 번째 커밋은, 예쁜 콘셉트 아트가 아니라 **내가 걸어 다니는 작은 야외**다.

<!-- ADSENSE: end -->

---

참고: 이전 단계([걷기·카메라](https://moyeo.blogspot.com/2026/08/unity-wow-rpg-ai.html))가 끝난 프로젝트를 전제로 한다. Terrain·URP·Lighting 메뉴 경로는 Unity 버전에 따라 조금 다를 수 있다. WoW 및 관련 명칭은 Blizzard 상표이며, 이 글은 조작·존 구성 학습용 메모다.
