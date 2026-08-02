# 카테고리(라벨) 가이드 — moyeo

Blogger의 **라벨**을 카테고리처럼 씁니다. 글당 주요 카테고리 1개를 붙입니다.

| 라벨 | 용도 | 예시 |
|------|------|------|
| **Travel** | Korea Spotlights / 한국 관광지 영어 가이드 | 노고단, 광명동굴, 울산바위 |
| **잡학** | 일상·사유·코드 비유 에세이 | 비 오는 날 기억, 시작 글 |
| **주식** | 시황·시장 구조 잡학 노트 | 서킷브레이커 |
| **개발 노트** | 코딩·개발 과정 메모·학습 기록 | — |

## 표시

사이드바는 HTML 위젯 **카테고리** 하나로 통일합니다. (라벨 가젯 `#Label1` + 구독 가젯을 합침)

- 카테고리명 → 라벨 목록 페이지
- 오른쪽 **RSS** → 해당 피드
- 상단 **이메일로 새 글 받기** → Blogtrottr(전체 글)

글 목록·본문 라벨 노출은 테마 설정을 유지합니다.

## 작성 시

1. Blogger 글 설정 → 라벨에 위 목록 중 하나 입력
2. 로컬 초안 메타에 `카테고리: 잡학` 형식으로도 남겨 둡니다

## 구독 (이메일 + RSS)

### 이메일

Blogger 기본 `Follow by Email` 가젯은 2021년 종료됐습니다. 전체 글 알림은 [Blogtrottr](https://blogtrottr.com/?subscribe=https%3A%2F%2Fmoyeo.blogspot.com%2Ffeeds%2Fposts%2Fdefault)(RSS→이메일)로 연결합니다. 카테고리별 이메일은 외부 ESP가 필요합니다.

### RSS / 라벨 페이지

| 구분 | 목록 | 피드 |
|------|------|------|
| **전체 글** | https://moyeo.blogspot.com/ | https://moyeo.blogspot.com/feeds/posts/default |
| Travel | /search/label/Travel | /feeds/posts/default/-/Travel |
| 잡학 | /search/label/잡학 | /feeds/posts/default/-/잡학 |
| 주식 | /search/label/주식 | /feeds/posts/default/-/주식 |
| 개발 노트 | /search/label/개발%20노트 | /feeds/posts/default/-/개발%20노트 |

### Blogger에 사이드바 위젯 적용

1. **레이아웃** → HTML/JavaScript 위젯(제목: `카테고리`)
2. 본문에 [`theme/category-subscribe.html`](../theme/category-subscribe.html) 내용(주석 아래 본문)을 붙여 넣습니다. 스타일은 가젯에 포함됩니다(`#Label1` 숨김 포함).
3. 기존 라벨 가젯·별도 구독 가젯이 있으면 제거하거나, CSS로 숨긴 채 둡니다.
4. 글 보관함 위에 배치한 뒤 저장합니다.
5. 로컬 미리보기: `theme/_cat-test.html`

Feedly, Inoreader, NetNewsWire 등 RSS 리더에 위 피드 URL을 넣으면 전체 글 또는 해당 카테고리만 받을 수 있습니다.
