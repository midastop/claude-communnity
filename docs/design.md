# 디자인 분석 (Figma)

| 항목 | 내용 |
| --- | --- |
| 문서 버전 | 0.1 (초안) |
| 작성일 | 2026-10-06 |
| 출처 | [Figma 파일 · Page 1](https://www.figma.com/design/R3s2lCuzGhYND5CrI2u1Ew/%25EC%25A0%259C%25EB%25AA%25A9-%25EC%2597%2586%25EC%259D%258C?node-id=0-1) |
| 기준 프레임 | 390 × 844 (모바일) |
| 관련 문서 | [요구사항 정의서](./requirements.md) |

## 0. 확인 수준

이 문서의 값은 출처에 따라 믿을 수 있는 정도가 다르다. 각 표에 아래 세 가지로 표시했다.

| 표시 | 뜻 | 해당 범위 |
| --- | --- | --- |
| **확정** | Figma 레이어의 스타일 값을 직접 읽었다 | 색 팔레트 전체, Button 섹션 대부분 |
| **배치** | 레이어 이름·위치·크기에서 계산했다. 크기와 간격은 정확하지만 색·글자·모서리는 알 수 없다 | 화면 12개, Navigation · Input · Feed · Setting 컴포넌트 |
| **미확인** | 읽지 못했다 | [8. 미확인 항목](#8-미확인-항목) |

Figma MCP 호출 한도(Starter 플랜, 월 20회)에 걸려 Navigation · Input · Feed · Setting 컴포넌트의 스타일과 화면 렌더는 읽지 못했다. 화면이 실제로 어떻게 보이는지는 이 문서로 확인되지 않는다.

---

## 1. 파일 구성

Page 1 한 장에 네 묶음이 놓여 있다.

| 묶음 (캔버스 제목) | 내용 |
| --- | --- |
| 색상 | `Color` 섹션 — Orange · Red · Gray 팔레트 |
| 이미지 | `Asset` 섹션 — logo, default-avatar |
| 컴포넌트 | `Navigation` · `Button` · `Input` · `Feed` · `Setting` 섹션 |
| 스크린 | 화면 12개. 인증 / 피드 / 프로필 / 설정 순으로 4줄 |

---

## 2. 화면 구조

### 2.1 공통 뼈대 (배치)

```
┌────────────────────────┐ y 0
│ StatusBar          44  │
├────────────────────────┤ 44
│ Header             59  │
├────────────────────────┤ 103
│                        │
│ 본문 (좌우 여백 16,     │
│       콘텐츠 폭 358)    │
│                        │
├────────────────────────┤ 762 또는 754
│ BottomTab 82           │
│  또는 FixedBottomCTA 90 │
└────────────────────────┘ 844
```

- 하단은 화면 성격에 따라 셋 중 하나다.
  - 탭의 첫 화면(Home · My · Setting): `BottomTab` 82
  - 입력 후 제출하는 화면(Signup · Login · Profile/Update): `FixedBottomCTA` 90
  - 그 외: 화면 전용 바(댓글 입력 바, 글쓰기 Footer) 또는 `SafeArea/Bottom` 34
- 피드가 있는 화면은 본문 뒤에 배경 사각형(`Rectangle 372`)이 깔리고, 카드 사이 12 간격으로 배경이 비친다.

### 2.2 화면 목록 (배치)

| Figma 프레임 | 요구사항 화면 | 상단 | 본문 | 하단 |
| --- | --- | --- | --- | --- |
| SplashScreen | 없음 | — | logo | — |
| AuthHome | 없음 | Header | logo, Filled/Large 버튼, "이메일로 가입하기" 텍스트 링크 | — |
| Signup | S-09 | Header | 이메일 · 비밀번호 · 비밀번호 확인 | FixedBottomCTA |
| Login | S-08 | Header | 이메일 · 비밀번호 | FixedBottomCTA |
| Home | S-01 | FeedHeader | FeedItem 목록, WriteButton | BottomTab |
| Feed/Detail | S-02 | Header + chevron-left | FeedItem, 댓글 | 댓글 입력 바 |
| Feed/Write | S-03 | Header + "저장" | 제목, 본문, 사진 썸네일 | FeedWrite Footer |
| Feed/Search | 없음 | arrow-left + 검색 입력 | FeedItem 목록, WriteButton | — |
| My | S-04 | 커버 이미지 | 아바타, 프로필 편집, ProfileDetail, Tabs, FeedItem | BottomTab |
| Profile/Update | S-05 | Header | 아바타, 사진 변경, 닉네임, 소개 | FixedBottomCTA |
| Profile/[id] | S-06 | 커버 이미지 + CustomHeader | 아바타, ProfileDetail, Tabs, FeedItem | SafeArea/Bottom |
| Setting | S-07 | Header | ListItem "로그아웃" | BottomTab |

### 2.3 화면별 배치 (배치)

좌표는 각 프레임의 왼쪽 위를 (0, 0)으로 한 값이다.

**SplashScreen**
- logo 112×112가 화면 정중앙 (x 139, y 366)

**AuthHome**
- Header (y 44)
- logo 112×112 (x 140, y 268)
- Filled/Large 버튼 326×44 (x 32, y 599) — 좌우 여백 32. 버튼 라벨은 미확인
- 버튼 아래 20 띄우고 "이메일로 가입하기" 텍스트 링크 (y 663, 가운데)

**Signup · Login**
- 필드 하나는 358×68 = 라벨 24 + 입력창 44
- 첫 필드 y 119 (Header 아래 16), 필드 사이 16 (y 119 → 203 → 287)
- Signup은 이메일 · 비밀번호 · 비밀번호 확인, Login은 이메일 · 비밀번호
- FixedBottomCTA (y 754)

**Home**
- FeedHeader 390×60 (y 44)
- FeedItem 390×361 (y 116, y 489) — 헤더와 카드, 카드와 카드 사이 12
- 카드 안 액션 줄은 왼쪽 16, 카드 위에서 320 (아래 여백 약 17)
- WriteButton 64×64 (x 310, y 682) — 오른쪽 16, BottomTab 위 16
- BottomTab (y 762)

**Feed/Detail**
- Header (y 44) + chevron-left 24×24 (x 16)
- FeedItem 390×361 (y 115)
- CommentHeader 390×48 (y 488, 카드 아래 12), 그 아래 댓글 목록
- 댓글 입력 바 390×110 = 위 16 + 입력창 358×44 + 아래 16 + 안전 영역 34
- 입력창 오른쪽 안에 Filled/Small 버튼 41×28 (오른쪽 7, 위아래 8)
- 긴 화면으로 그려져 있다. 입력 바(y 825)와 키보드(y 901)가 프레임 높이 844를 넘어 이어진다

**Feed/Write**
- Header (y 44) + 오른쪽 "저장" 텍스트 버튼 (Standard/Medium 39×20, 오른쪽 16)
- 제목 필드 358×68 (y 119)
- 본문 라벨 (y 203) + 여러 줄 입력 358×188 (y 227)
- 사진 썸네일 90×90 (y 431, 본문 아래 16), 썸네일 사이 5
- FeedWrite Footer 390×62 (y 782, 화면 맨 아래)

**Feed/Search**
- 헤더 자리(y 44, 높이 60)에 arrow-left 24×24 (x 15) + 검색 입력 324×44 (x 50, y 52). placeholder "글 제목 검색"
- 아래는 Home과 같은 피드 + WriteButton
- 키보드 390×291 (y 556)

**My**
- 커버 이미지 390×154 (y 0, 상태바 뒤까지 채움)
- 아바타 152×152 (x 16, y 78) — 위 절반은 커버에, 아래 절반은 커버 밖에 걸친다
- "프로필 편집" Outlined/Medium 98×38 (x 276, y 100) — 커버 위 오른쪽, 오른쪽 여백 16
- ProfileDetail 390×112 (y 230)
- Tabs 390×38 (y 336)
- FeedItem (y 386, 탭 아래 12)
- BottomTab (y 762)

**Profile/[id]**
- My와 같은 뼈대에서 "프로필 편집" 버튼과 BottomTab이 빠진다
- 커버 위에 CustomHeader 390×56 (y 0)이 겹친다
- Tabs (y 337), FeedItem (y 374)
- 하단은 SafeArea/Bottom 34 (y 810)

**Profile/Update**
- Header (y 44)
- 아바타 152×152 가운데 (x 119, y 135 — Header 아래 32)
- "사진 변경" Outlined/Medium 98×38 (x 276, y 248) — 아바타 오른쪽 아래
- 닉네임 필드 358×68 (y 287), 소개 필드 (y 371, 간격 16)
- FixedBottomCTA (y 754)

**Setting**
- Header (y 44)
- ListItem 390×40 (y 133, Header 아래 30) — "로그아웃", 왼쪽 14, 위아래 12
- BottomTab (y 762)

> **파일 정리 상태** — My · Setting · Profile/Update · Feed/Write · Feed/Detail의 일부 레이어(커버, 아바타, ListItem, 입력 필드, Footer, 키보드)는 프레임 안이 아니라 페이지에 직접 놓여 프레임 위에 겹쳐져 있다. 프레임 단위로 내보내면 이 레이어들이 빠진다.

---

## 3. 주요 컴포넌트

### 3.1 Navigation (배치)

| 컴포넌트 | 변형 | 크기 | 쓰이는 화면 |
| --- | --- | --- | --- |
| StatusBar | — | 390×44 | 전 화면 |
| Header | Default · TwoButton | 390×59 | AuthHome · Signup · Login · Feed/Detail · Feed/Write · Profile/Update · Setting (어느 변형인지는 미확인) |
| Header | CustomHeader · CustomHeader/Black | 390×56 | Profile/[id] (커버 위) |
| BottomTab | — | 390×82 | Home · My · Setting |
| Tabs | Two · Three · Four | 390×38 | My · Profile/[id] |
| SafeArea/Bottom | — | 390×34 | Profile/[id], FixedBottomCTA 내부 |

### 3.2 Button

| 이름 | 크기 | 모양 | 글자 | 비활성 | 수준 |
| --- | --- | --- | --- | --- | --- |
| Standard/Small | 23×20 | 배경 없음 (텍스트 버튼) | 12 / 줄높이 20, `#FF6B57` | 글자 `#C1C1C1` | 확정 |
| Standard/Medium | 39×20 | — | — | — | 미확인 |
| Filled/Small | 40×28 | 배경 `#FF6B57`, 모서리 6 | 12 / 20, 흰색 | 회색 채움 (렌더로만 확인) | 확정 |
| Outlined/Small | 40×28 | 흰 배경 + 1px `#FF6B57`, 모서리 4 | 12 / 20, `#FF6B57` | 회색 테두리·글자 (렌더로만 확인) | 확정 |
| Filled/Medium | 50×38 | 배경 `#FF6B57`, 모서리 8, 패딩 위 9 · 아래 8 · 좌우 12 | 14, 흰색 | 배경 `#D1D5DB` | 확정 |
| Outlined/Medium | 50×38 (화면에서는 98×38) | — | — | — | 미확인 |
| Filled/Large | 326×44 | Filled/Medium과 같음 | 14, 흰색 | 배경 `#D1D5DB` | 확정 |
| WriteButton | 64×64 | 플로팅 글쓰기 버튼 | — | — | 미확인 |
| FixedBottomCTA | 390×90 | 아래 참고 | | | 확정 |
| Icon | 24×24 | 선으로 그린 아이콘 10종 | | | 확정 |

**FixedBottomCTA** (확정)
- 흰 배경, 위쪽에 1px `#EBEBEB` 구분선
- 위 12 + Filled/Large 버튼(좌우 16, 358×44) + SafeArea/Bottom 34
- 홈 인디케이터: 134×5, 검정, 모서리 100, 아래에서 8

**Icon** (확정)
- Like(heart) · Comment(message-circle) · Share · Cancel(원 안의 X) · Reply(꺾인 화살표) · Camera · Vote(상자) · Setting(톱니) · eye-outline · User
- 채움 없는 선 아이콘. 레이어 이름이 Feather 아이콘 이름과 같고, eye만 ionicons다
- 색 스타일은 `main/dark #232323`, `Slate/900 #0F172A` 두 가지가 섞여 있다

### 3.3 Input (배치)

| 묶음 | 변형 | 크기 |
| --- | --- | --- |
| Filled | Input | 358×44 |
| Filled | MultiLine | 358×85 (Feed/Write에서는 188로 늘려 씀) |
| Filled | InputField | 358×68 = 라벨 24 + 입력창 44 |
| Filled | InputField / Error | 358×92 = InputField + 오류 문구 24 |
| Search | SearchInput | 306×44 (Feed/Search에서는 324) |
| Outlined | Default · Icon | 358×44 |
| Standard | Default · Icon | 358×44 |

- 화면에 실제로 쓰인 것은 Filled와 Search뿐이다. Outlined · Standard는 컴포넌트만 있다.
- 입력창 안 글자의 왼쪽 여백은 레이어마다 10 · 11 · 13 · 16으로 다르다. 구현할 때 하나로 정해야 한다.

### 3.4 Feed (배치)

| 묶음 | 변형 | 크기 |
| --- | --- | --- |
| FeedItem | FeedHeader | 390×60 |
| FeedItem | Basic | 390×231 |
| FeedItem | SinglePicture · MultiplePicture | 390×361 (Basic + 130) |
| Picture | PictureItem | 90×90 |
| Action | Like · Comment · eye | 47×24 (아이콘 24 + 숫자) |
| Profile | Avatar (Feed) | 45×45 |
| Profile | FeedItemProfile · CommentItemProfile | 358×46 |
| Profile | Follow (True · False) | 358×46 |
| Profile | ProfileDetail | 390×112 |
| Comment | CommentHeader | 390×48 |
| Comment | CommentInput | 390×76 |
| Comment | Enroll | 390×110 (CommentInput + 안전 영역 34) |
| Comment | Default | 390×154 |
| FeedWrite | Footer | 390×62 |

### 3.5 Setting · Asset (배치)

| 컴포넌트 | 크기 | 비고 |
| --- | --- | --- |
| ListItem | 390×40 | Setting 화면의 "로그아웃" |
| ActionSheet | 365×134 | 어느 화면에도 놓여 있지 않다 |
| logo | 112×112 | SplashScreen · AuthHome |
| default-avatar | 229×229 | 화면에서는 152 · 45로 줄여 쓴다 |

> **이름 정리** — Figma 변형 이름이 정돈돼 있지 않다 (`View=View2`가 Gray/600과 비활성 버튼 양쪽에 쓰임, `Stardard` · `Filed` 오타, `Property 2`). 코드에서는 Figma 이름을 옮기지 말고 새로 이름을 붙인다.

---

## 4. 색

### 4.1 팔레트 (확정)

| 계열 | 단계 | 값 | Tailwind 기본 색과의 관계 |
| --- | --- | --- | --- |
| Orange | 100 | `#FFF7F1` | — |
| Orange | 200 | `#FFDEC6` | — |
| Orange | 300 | `#FFB884` | — |
| Orange | 600 | `#FF6B57` | — |
| Red | 100 | `#FFDFDF` | — |
| Red | 500 | `#FF5F5F` | — |
| Gray | 100 | `#F6F6F6` | — |
| Gray | 200 | `#E2E8F0` | slate-200 |
| Gray | 300 | `#D1D5DB` | gray-300 |
| Gray | 500 | `#6B7280` | gray-500 |
| Gray | 600 | `#4B5563` | gray-600 |
| Gray | 700 | `#374151` | gray-700 |

- 단계가 이어지지 않는다. Orange는 300에서 600으로 건너뛰고, Gray는 400이 없다.
- Figma 변수(Variables)는 조회 결과가 비어 있었다. 색은 변수로 묶여 있지 않고 값으로 들어가 있다.

### 4.2 쓰임

| 역할 | 값 | 근거 | 수준 |
| --- | --- | --- | --- |
| 주색 | Orange/600 `#FF6B57` | 채움 버튼 배경, Outlined 테두리·글자, 텍스트 버튼 글자 | 확정 |
| 주색 위 글자 | `#FFFFFF` | 채움 버튼 라벨 | 확정 |
| 표면 | `#FFFFFF` | FixedBottomCTA, Outlined 버튼 배경 | 확정 |
| 비활성 채움 | Gray/300 `#D1D5DB` | Filled/Medium · Large 비활성 | 확정 |
| 비활성 글자 | `#C1C1C1` | Standard/Small 비활성. 팔레트에 없는 값 | 확정 |
| 구분선 | `#EBEBEB` | FixedBottomCTA 위 테두리. 팔레트에 없는 값 | 확정 |
| 아이콘 | `#232323`, `#0F172A` | Icon 프레임의 색 스타일 | 확정 |
| 오류 | Red/500 `#FF5F5F` | InputField / Error에 쓰였을 것으로 추정 | 미확인 |
| 화면 배경 | Gray/100 `#F6F6F6` | 피드 뒤 배경 사각형에 쓰였을 것으로 추정 | 미확인 |
| Orange 100 ~ 300, Red/100 | — | 쓰인 곳을 확인하지 못함 | 미확인 |

### 4.3 톤

- 흰 바탕에 중성 회색, 강조색은 코럴 오렌지 하나다. 확인한 범위에서 주색 외에 색을 쓰는 곳은 없다.
- 회색은 푸른 기가 도는 차가운 계열(Tailwind gray · slate)이고, 강조색과 그 옅은 단계는 따뜻한 계열이다.
- 비활성은 색을 빼고 회색으로만 표현한다. 주색을 옅게 만드는 방식이 아니다.

---

## 5. 글자

| 쓰임 | 글꼴 | 크기 / 줄높이 | 굵기 | 수준 |
| --- | --- | --- | --- | --- |
| Small 버튼 라벨 | IBM Plex Sans | 12 / 20 | SemiBold (600) | 확정 |
| Medium · Large 버튼 라벨 (스타일 이름 `body/Small/bold`) | Poppins | 14 / 100% | SemiBold (600) | 확정 |
| 입력 필드 라벨 | — | 약 12 / 24 | — | 추정 |
| 입력창 글자 · placeholder | — | 약 14 / 24 | — | 추정 |
| ListItem 글자 | — | 약 12 / 16 | — | 추정 |

- "추정"은 글자 상자의 폭과 높이에서 역산한 값이다. 줄높이(상자 높이)는 정확하고 글자 크기는 근사치다.
- Poppins와 IBM Plex Sans에는 한글 글리프가 없다. Figma에서 한글은 대체 글꼴로 그려지고 있으므로, 구현할 때 한글 글꼴을 따로 정해야 한다.
- 제목 · 본문 등 나머지 글자 스타일은 확인하지 못했다.

---

## 6. 간격 · 크기 톤

### 6.1 간격

| 값 | 쓰이는 곳 | 수준 |
| --- | --- | --- |
| 5 | 사진 썸네일 사이 | 배치 |
| 8 | 검색 헤더 위아래 여백, 댓글 입력창 안 버튼 여백 | 배치 |
| 12 | 피드 카드 사이, 헤더와 첫 카드 사이, 탭과 첫 카드 사이, ListItem 위아래, FixedBottomCTA 위 여백, 버튼 좌우 패딩 | 배치 · 확정 |
| 16 | 화면 좌우 여백, 폼 필드 사이, Header와 첫 필드 사이, 댓글 입력 바 위아래, WriteButton 오른쪽·아래 | 배치 |
| 20 | AuthHome 버튼과 텍스트 링크 사이 | 배치 |
| 32 | AuthHome 버튼 좌우 여백, Profile/Update의 Header와 아바타 사이 | 배치 |
| 34 | 하단 안전 영역 | 확정 |

기본 단위는 **16**(화면 여백, 폼)과 **12**(피드 목록)다. 폼 화면은 16으로 넉넉하게, 피드는 12로 촘촘하게 쓴다.

### 6.2 높이

| 값 | 쓰이는 곳 |
| --- | --- |
| 24 | 아이콘, 글자 한 줄 |
| 28 | Small 버튼 |
| 38 | Medium 버튼, Tabs |
| 40 | ListItem |
| 44 | 입력창, Large 버튼, StatusBar |
| 56 · 59 | Header (Custom · 기본) |
| 60 | FeedHeader |
| 62 | FeedWrite Footer |
| 76 | 댓글 입력 바 (16 + 44 + 16) |
| 82 | BottomTab |
| 90 | FixedBottomCTA (12 + 44 + 34) |

### 6.3 모서리 (확정)

| 값 | 쓰이는 곳 |
| --- | --- |
| 4 | Outlined/Small |
| 6 | Filled/Small |
| 8 | Filled/Medium · Large |
| 100 | 홈 인디케이터 (알약 모양) |

입력창 · 카드 · 썸네일 · 아바타의 모서리는 확인하지 못했다.

### 6.4 그 밖의 크기

| 항목 | 값 |
| --- | --- |
| 콘텐츠 폭 | 358 (390 − 16 × 2) |
| 아바타 | 45 (피드 · 댓글), 152 (프로필) |
| 커버 이미지 | 390×154 |
| 사진 썸네일 | 90×90 |
| 플로팅 버튼 | 64×64 |

---

## 7. 요구사항 정의서와 다른 점

디자인과 [요구사항 정의서](./requirements.md)가 서로 다른 부분이다. 어느 쪽을 따를지는 정해지지 않았다.

### 7.1 디자인에만 있는 것

| 항목 | 근거 레이어 | 요구사항 |
| --- | --- | --- |
| 글 검색 | `Feed/Search` 화면, SearchInput | 0.3에서 제외 |
| 글 사진 첨부 | PictureItem, FeedItem의 SinglePicture · MultiplePicture, Camera 아이콘, Feed/Write의 썸네일 | 0.3에서 제외 (가정 3) |
| 팔로우 | Profile의 `Follow` True · False | 0.3에서 제외 |
| 답글 | Reply 아이콘 | 0.3에서 제외 (대댓글) |
| 공유 · 투표 · 조회수 | Share · Vote 아이콘, Action의 `eye` | 언급 없음 |
| 자기소개 | Profile/Update의 "소개" 필드 | PROF-02에 없음 |
| 커버 이미지, 프로필 탭 | cover image, Tabs | PROF-01 · 03에 없음 |
| 스플래시, 인증 홈 | `SplashScreen`, `AuthHome` 화면 | 화면 목록에 없음 |
| 액션 시트 | ActionSheet 컴포넌트 (용도 미확인) | 언급 없음 |

### 7.2 요구사항에만 있는 것

| 항목 | 디자인 상태 | 요구사항 |
| --- | --- | --- |
| 회원가입 닉네임 입력 | Signup에 이메일 · 비밀번호 · 비밀번호 확인만 있다 | AUTH-01 |
| 설정의 계정 이메일, 앱 버전 | Setting에 "로그아웃" 항목 하나만 있다 | SET-01 |
| 비회원용 화면 (로그인 안내, "로그인하고 댓글 쓰기") | 해당 프레임 없음 | PROF-01, CMT-02 |
| 빈 상태 · 불러오는 중 · 오류 화면 | 해당 프레임 없음 | POST-02, POST-03, PROF-03 |
| 로그아웃 확인 창 | 해당 프레임 없음 (ActionSheet가 이 용도일 수 있음) | AUTH-03 |

### 7.3 수치가 어긋나는 것

| 항목 | 디자인 | 요구사항 |
| --- | --- | --- |
| 누르는 영역 | Small 버튼 28, Medium 버튼 · Tabs 38, ListItem 40, 아이콘 24 | 최소 44×44 (2.7) |

보이는 크기는 디자인대로 두고 누르는 영역만 44로 넓히면 둘 다 만족한다.

---

## 8. 미확인 항목

아래는 읽지 못해 이 문서에 값이 없다. 구현 전에 Figma에서 확인해야 한다.

| 영역 | 확인할 것 |
| --- | --- |
| Navigation | BottomTab의 탭 개수 · 아이콘 · 라벨 · 선택 색, Header 4종의 내용과 색, Tabs의 선택 표시 |
| Input | 배경색, 테두리, 모서리, placeholder 색, Error 상태의 색과 문구 스타일 |
| Feed | FeedItem 내부 배치와 글자 크기, 댓글 항목, ProfileDetail 내용 |
| Setting | ListItem · ActionSheet의 스타일과 내용 |
| Button | Standard/Medium, Outlined/Medium, WriteButton의 스타일. Small 버튼 비활성의 정확한 값 |
| 화면 | 배경색, AuthHome 버튼 라벨, 각 화면의 헤더 제목. 화면 렌더를 보지 못했으므로 2장의 배치가 실제 모습과 맞는지 |
| 글자 | 제목 · 본문 · 캡션 등 글자 스타일 전반, 한글 글꼴 |
