# Unity로 WoW를 닮은 RPG — AI와 첫 질문부터 걷기까지

> 상태: published  
> URL: https://moyeo.blogspot.com/2026/08/unity-wow-rpg-ai.html  
> 작성일: 2026-08-04  
> 카테고리: 개발 노트  
> 주제: Unity + AI로 WoW식 3인칭 RPG — 범위 질문·씬 구성·CharacterController 이동

“와우 같은 거 만들고 싶어”는 목표가 아니라 **지평선**이다.  
퀘스트, 직업, 레이드, UI, 서버… 한 번에 열면 AI도, 나도 길을 잃는다.

오늘은 목표를 **Unity 3인칭 + WoW식 카메라/이동**으로 자른다.  
AI에게 던질 **첫 질문**부터, 캐릭터가 맵 위를 **걷고 카메라가 따라오는** 순간까지만 남긴다.  
전투·인벤·퀘스트는 다음 로그다.

## 전제: “와우 같다”를 동작으로 번역한다

와우를 닮게 만드는 첫 체감은 스토리가 아니라 **조작감**이다.

| WoW에서 느끼는 것 | Unity에서 구현할 것 (1단계) |
|------------------|---------------------------|
| 캐릭터 뒤에 카메라 | 3인칭 오빗/팔로우 카메라 |
| WASD로 이동 | 카메라 기준 수평 이동 |
| 마우스로 시점 | yaw/pitch로 카메라 회전 |
| 지형 위 걷기 | `CharacterController` + 중력 |
| (나중) 타겟팅·스킬 | 이번 글 범위 밖 |

이 표를 AI 대화 맨 위에 붙여 두면, “오픈월드 MMO 전체”로 새어 나가는 걸 줄일 수 있다.

<!-- diagram: wow-like loop -->
```text
[입력 WASD/마우스] → [이동·회전] → [카메라 추적] → [화면]
         ↑__________________________________|
              (매 프레임 반복하는 루프만 먼저)
```

## 1) AI에게 던질 첫 질문 (복붙용)

새 채팅을 열자마자, 아래를 **한 덩어리**로 보낸다.  
막연한 “와우 만들어줘” 대신 **스펙**이다.

```text
목표: Unity 6(또는 2022 LTS)로 WoW 비슷한 3인칭 RPG의 "이동+카메라"만 만든다.
범위 밖: 멀티플레이, 전투, 인벤, 퀘스트, 애니메이션 고급 블렌드.
기술 선택:
- 렌더 파이프라인: URP
- 이동: CharacterController
- 입력: 새 Input System (또는 Legacy 명시)
- 카메라: 캐릭터 자식이 아닌, 별도 CameraRig가 캐릭터를 따라감
- 시점: 마우스 우클릭 드래그로 yaw/pitch (또는 항상 자유 시점 — 너는 우클릭 방식 권장안을 제시)
산출물:
1) 권장 씬 하이어라키(노드 이름 포함)
2) PlayerMover.cs, CameraOrbit.cs 전체 코드
3) Inspector에서 넣을 값 가이드
4) Play 모드에서 확인할 체크리스트 5줄
제약: 설명은 짧게, 코드는 복사 가능하게. 매 답변 끝에 "다음에 넣을 기능 후보 1개"만.
```

이 질문이 좋은 이유:

- **엔진·버전·범위**가 고정된다  
- AI가 서버부터 설계하지 못한다  
- “다음에 넣을 기능 1개”로 대화가 비대해지지 않는다  

답이 오픈월드 전체로 새면, 한 줄만 더 보낸다.

```text
범위 밖이다. CharacterController 이동과 오빗 카메라만 다시 줘.
```

## 2) Unity 프로젝트 뼈대

로컬에서 할 일 (AI에게 “클릭 경로”를 물어봐도 된다).

| 단계 | 할 일 |
|------|------|
| 1 | Unity Hub → 3D (URP) 템플릿으로 프로젝트 생성 |
| 2 | 바닥: Plane 또는 간단한 Terrain |
| 3 | 플레이어: Capsule + `CharacterController` |
| 4 | 카메라: Main Camera를 플레이어 자식에서 **빼기** |
| 5 | 빈 오브젝트 `CameraRig` 생성 후 스크립트 연결 |

권장 하이어라키:

```text
Plane (바닥)
Player
  ├─ Capsule (메시)
  └─ CharacterController
CameraRig
  └─ Pivot
       └─ Main Camera
Directional Light
```

관계를 그림으로 보면 이렇게다.

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 220" width="100%" style="max-width:640px;height:auto;margin:1em 0;background:#f3f0e8;border-radius:4px;">
  <rect x="40" y="130" width="120" height="50" fill="#2f6b4f" rx="4"/>
  <text x="100" y="160" text-anchor="middle" fill="#f3f0e8" font-size="14" font-family="sans-serif">Player</text>
  <rect x="260" y="40" width="120" height="50" fill="#1a2420" rx="4"/>
  <text x="320" y="70" text-anchor="middle" fill="#f3f0e8" font-size="14" font-family="sans-serif">CameraRig</text>
  <rect x="260" y="120" width="120" height="50" fill="#3d4a44" rx="4"/>
  <text x="320" y="150" text-anchor="middle" fill="#f3f0e8" font-size="14" font-family="sans-serif">Pivot</text>
  <rect x="460" y="120" width="140" height="50" fill="#6b7872" rx="4"/>
  <text x="530" y="150" text-anchor="middle" fill="#f3f0e8" font-size="14" font-family="sans-serif">Main Camera</text>
  <path d="M160 155 H250" stroke="#1a2420" stroke-width="2" fill="none" marker-end="url(#a)"/>
  <path d="M320 90 V120" stroke="#1a2420" stroke-width="2" fill="none"/>
  <path d="M380 145 H460" stroke="#1a2420" stroke-width="2" fill="none"/>
  <text x="200" y="145" fill="#3d4a44" font-size="11" font-family="sans-serif">follow</text>
  <defs><marker id="a" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="#1a2420"/></marker></defs>
  <text x="40" y="30" fill="#1a2420" font-size="13" font-family="sans-serif">WoW식: 카메라는 Player 자식이 아니라 Rig가 따라간다</text>
</svg>

**함정:** Main Camera를 Capsule 자식으로 두면 “몸과 시점이 한 몸”이 되어, 와우식 오빗이 불편해진다.  
AI가 자식으로 붙여 주면 교정 프롬프트를 보낸다.

```text
카메라는 Player 자식이 아니야. CameraRig가 Player 위치를 따라가고,
Pivot에서 yaw/pitch 회전하는 구조로 바꿔줘.
```

<!-- ADSENSE: mid -->

## 3) 이동 스크립트 — CharacterController

아래는 **최소 동작** 예시이다. AI에게 생성시킨 뒤, 이 구조와 비교해 보면 된다.  
(프로젝트 Input 설정에 따라 `Input.GetAxis` 대신 새 Input System으로 바꿔 달라고 요청하면 된다.)

```csharp
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class PlayerMover : MonoBehaviour
{
    public float moveSpeed = 5f;
    public float turnSpeed = 12f;
    public float gravity = -20f;
    public Transform cameraTransform; // Main Camera

    CharacterController _cc;
    float _verticalVel;

    void Awake() => _cc = GetComponent<CharacterController>();

    void Update()
    {
        float h = Input.GetAxisRaw("Horizontal"); // A/D
        float v = Input.GetAxisRaw("Vertical");   // W/S

        // 카메라 기준 수평 방향 (Y 제거)
        Vector3 camForward = cameraTransform.forward;
        Vector3 camRight = cameraTransform.right;
        camForward.y = 0f;
        camRight.y = 0f;
        camForward.Normalize();
        camRight.Normalize();

        Vector3 move = (camForward * v + camRight * h).normalized;

        if (move.sqrMagnitude > 0.001f)
        {
            Quaternion look = Quaternion.LookRotation(move);
            transform.rotation = Quaternion.Slerp(
                transform.rotation, look, turnSpeed * Time.deltaTime);
        }

        if (_cc.isGrounded && _verticalVel < 0f)
            _verticalVel = -2f;
        _verticalVel += gravity * Time.deltaTime;

        Vector3 velocity = move * moveSpeed;
        velocity.y = _verticalVel;
        _cc.Move(velocity * Time.deltaTime);
    }
}
```

핵심만 짚으면:

- 이동 벡터는 **월드 축이 아니라 카메라 forward/right**  
- 캐릭터 몸체는 이동 방향을 바라보게 `Slerp`  
- 중력은 `CharacterController.Move`에 y로 합친다  

AI가 `Rigidbody` + `AddForce`로 주면, 와우식 “딱 붙는 걷기”와 결이 달라질 수 있다.  
원하면 이렇게 못 박는다.

```text
Rigidbody 말고 CharacterController로. 미끄러짐 없이 WASD 걷기로.
```

## 4) 카메라 오빗 — 우클릭으로 시점

와우 기본 감각에 가까운 쪽: **우클릭 드래그로 회전**, 카메라는 캐릭터 뒤를 유지.

```csharp
using UnityEngine;

public class CameraOrbit : MonoBehaviour
{
    public Transform target;      // Player
    public Transform pivot;       // Pivot
    public float distance = 5f;
    public float minPitch = -20f;
    public float maxPitch = 60f;
    public float sensitivity = 3f;
    public Vector3 targetOffset = new Vector3(0f, 1.6f, 0f);

    float _yaw;
    float _pitch = 15f;

    void LateUpdate()
    {
        if (target == null) return;

        if (Input.GetMouseButton(1)) // 우클릭
        {
            _yaw += Input.GetAxis("Mouse X") * sensitivity;
            _pitch -= Input.GetAxis("Mouse Y") * sensitivity;
            _pitch = Mathf.Clamp(_pitch, minPitch, maxPitch);
        }

        transform.position = target.position + targetOffset;
        pivot.rotation = Quaternion.Euler(_pitch, _yaw, 0f);

        // 카메라를 pivot 뒤로 distance만큼
        Camera cam = GetComponentInChildren<Camera>();
        if (cam != null)
            cam.transform.localPosition = new Vector3(0f, 0f, -distance);
    }
}
```

Inspector 연결:

| 필드 | 넣을 것 |
|------|---------|
| `PlayerMover.cameraTransform` | Main Camera |
| `CameraOrbit.target` | Player |
| `CameraOrbit.pivot` | Pivot |
| Capsule `Height/Center` | CharacterController와 맞게 |

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 200" width="100%" style="max-width:640px;height:auto;margin:1em 0;background:#f3f0e8;border-radius:4px;">
  <circle cx="280" cy="120" r="18" fill="#2f6b4f"/>
  <text x="280" y="125" text-anchor="middle" fill="#f3f0e8" font-size="11" font-family="sans-serif">P</text>
  <path d="M280 120 Q360 40 440 100" stroke="#1a2420" stroke-width="2" fill="none" stroke-dasharray="4 3"/>
  <rect x="420" y="80" width="70" height="40" fill="#6b7872" rx="3"/>
  <text x="455" y="105" text-anchor="middle" fill="#f3f0e8" font-size="12" font-family="sans-serif">Cam</text>
  <text x="40" y="36" fill="#1a2420" font-size="13" font-family="sans-serif">우클릭 드래그 → yaw/pitch → 카메라는 거리(distance) 유지</text>
  <text x="40" y="170" fill="#3d4a44" font-size="12" font-family="sans-serif">WASD는 "카메라가 보는 바닥 평면" 기준으로 이동</text>
</svg>

## 5) Play 모드 체크리스트

AI에게 코드를 받은 뒤, **실행 전에** 이 다섯 줄을 확인한다.

| # | 확인 |
|---|------|
| 1 | Player에 `CharacterController`가 있고, Capsule Collider와 충돌이 이중으로  overlapping하지 않는가 |
| 2 | Main Camera가 Player 자식이 **아닌가** |
| 3 | `cameraTransform` / `target` 참조가 비어 있지 않은가 |
| 4 | Plane에 Collider가 있는가 |
| 5 | Game 뷰 클릭 후 WASD·우클릭 드래그가 먹는가 |

막히면 에러 로그 전문을 그대로 붙인다.

```text
이 에러 기준으로 PlayerMover/CameraOrbit만 수정해줘. 새 시스템 추가 금지.
NullReferenceException: ...
```

## 6) “와우 비슷함”을 한 단계만 더

걷기가 되면, AI에게 **하나만** 요청한다. 예:

```text
다음 기능 하나만: 스페이스 점프 (CharacterController, 이단 점프 없음).
기존 WASD·카메라 코드는 깨지 마.
```

또는:

```text
다음 기능 하나만: 이동 중 Capsule이 카메라 방향이 아니라
이동 벡터를 보게 (이미 있으면 튜닝값만).
```

전투·타겟팅·네임플레이트를 한꺼번에 시키면, 어제 되던 이동이 오늘 깨지는 경우가 많다.  
와우를 닮은 RPG의 첫 커밋은 **맵 위를 내가 조종하는 아바타**면 충분하다.

## 손에 남길 최소 세트

| # | 완료 조건 |
|---|-----------|
| 1 | 첫 질문에 범위·URP·CharacterController·CameraRig를 못 박았다 |
| 2 | 하이어라키에서 카메라가 Player 자식이 아니다 |
| 3 | WASD가 카메라 기준으로 움직인다 |
| 4 | 우클릭으로 시점이 돈다 |
| 5 | 중력으로 바닥에 서 있다 |

완벽한 와우 클론은 목표가 아니다.  
**조작 루프 하나**가 Play 모드에서 돌아가면, AI와의 다음 대화가 비로소 “RPG 제작”이 된다.

<!-- ADSENSE: end -->

---

참고: Unity 버전·Input System·URP 설정은 프로젝트마다 다르다. 위 코드는 학습용 최소 골격이며, 공식 샘플(Starter Assets - Third Person)과 병행하면 디버깅이 빨라진다. WoW는 Blizzard 상표이며, 이 글은 조작 감각을 참고하는 개인 학습 메모다.
