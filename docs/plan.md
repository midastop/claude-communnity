# 모바일 웹 커뮤니티 개발 계획 (Next.js + Supabase)

| 항목 | 내용 |
| --- | --- |
| 문서 버전 | 0.1 (초안) |
| 작성일 | 2026-10-06 |
| 관련 문서 | [요구사항 정의서](./requirements.md) · [디자인 분석](./design.md) |

## 배경

- 저장소에는 `docs/requirements.md`(요구사항)와 `docs/design.md`(Figma 분석)만 있고 코드는 없다. 이 계획은 두 문서를 실제 구현 순서로 옮긴 것이다.
- **범위 결정**: 기능과 데이터는 요구사항 기준이다. 디자인에서는 색·간격·컴포넌트 모양만 가져온다.
  - 화면에서 빼는 것: 검색, 글 사진 첨부, 팔로우, 답글, 공유·투표·조회수, 자기소개, 커버 이미지, 프로필 탭, 스플래시, 인증 홈
  - 디자인에 없지만 넣는 것: 회원가입의 닉네임 입력, 설정의 이메일·앱 버전, 비회원 안내 화면, 빈 상태·로딩·오류 화면, 로그아웃 확인 창
- **디자인 미확인분**: 하단 탭, 입력창, 피드 카드 내부의 스타일은 Figma에서 읽지 못했다(`design.md` 8장). 확인된 팔레트와 간격 톤에 맞춰 먼저 만들고, Figma를 다시 읽을 수 있을 때 보정한다.

## 기술 선택

| 항목 | 선택 |
| --- | --- |
| 프레임워크 | Next.js 최신 안정판 (App Router, TypeScript, `src/` 사용) |
| 스타일 | Tailwind CSS v4 — `globals.css`의 `@theme`에 디자인 토큰 정의 |
| Supabase | `@supabase/supabase-js` + `@supabase/ssr` (쿠키 세션), Supabase CLI로 마이그레이션·타입 생성 |
| 조회 | 서버 컴포넌트가 첫 20개를 그리고, 다음 페이지는 브라우저 클라이언트가 같은 조회 함수로 불러온다 |
| 변경 | Server Actions (가입·로그인·로그아웃·글·댓글·좋아요·프로필) |
| 검증 | zod — 같은 규칙을 폼과 Server Action 양쪽에서 쓴다 |
| 아이콘 | lucide-react (디자인의 Feather 아이콘과 같은 계열) |
| 한글 글꼴 | Pretendard (디자인에 한글 글꼴이 정해져 있지 않아 가정) |

## 1. 폴더 구조

```
community/
├─ docs/                     requirements.md · design.md · plan.md
├─ supabase/
│  ├─ config.toml
│  └─ migrations/
│     ├─ 0001_profiles.sql          profiles + 가입 트리거 + RLS
│     ├─ 0002_posts.sql             posts + RLS + 인덱스
│     ├─ 0003_comments_likes.sql    comments · likes + RLS
│     └─ 0004_storage_avatars.sql   avatars 버킷 + 정책
├─ public/                   logo.svg · default-avatar.png
├─ src/
│  ├─ proxy.ts               세션 쿠키 갱신 + 접근 제어 (Next 15 이하면 middleware.ts)
│  ├─ app/
│  │  ├─ layout.tsx          최대 폭 480 가운데 정렬, 글꼴, viewport
│  │  ├─ globals.css         @theme 토큰
│  │  ├─ error.tsx · not-found.tsx
│  │  ├─ (tabs)/             하단 탭이 있는 화면
│  │  │  ├─ layout.tsx       BottomTab
│  │  │  ├─ page.tsx         S-01 홈
│  │  │  ├─ profile/page.tsx S-04 내정보
│  │  │  └─ settings/page.tsx S-07 설정
│  │  ├─ (stack)/            뒤로가기 헤더가 있는 화면
│  │  │  ├─ posts/new/page.tsx     S-03
│  │  │  ├─ posts/[id]/page.tsx    S-02 (+ not-found.tsx)
│  │  │  ├─ profile/edit/page.tsx  S-05
│  │  │  └─ users/[id]/page.tsx    S-06 (+ not-found.tsx)
│  │  └─ (auth)/
│  │     ├─ login/page.tsx   S-08
│  │     └─ signup/page.tsx  S-09
│  ├─ components/
│  │  ├─ ui/        Button · TextField · TextArea · Avatar · Header · BottomTab
│  │  │             FixedBottomCTA · ConfirmDialog · Skeleton · EmptyState · ErrorState
│  │  ├─ post/      PostCard · PostList(무한 스크롤) · PostForm · LikeButton
│  │  ├─ comment/   CommentList · CommentItem · CommentForm
│  │  └─ profile/   ProfileHeader · ProfileForm · AvatarPicker
│  └─ lib/
│     ├─ supabase/  client.ts · server.ts · proxy.ts · database.types.ts(생성)
│     ├─ actions/   auth.ts · posts.ts · comments.ts · likes.ts · profile.ts
│     ├─ queries/   posts.ts · comments.ts · profiles.ts
│     ├─ auth.ts        getUser() · requireUser(next)
│     ├─ validation.ts  닉네임·비밀번호·글자 수 규칙
│     └─ format.ts      상대 시간 표시
├─ .env.example · .env.local(커밋 안 함)
└─ package.json
```

- 라우트 그룹 `(tabs)` · `(stack)` · `(auth)`는 주소에 나타나지 않는다. 하단 탭 유무(NAV-01)를 레이아웃으로 가르기 위한 것이다.
- `lib/queries`의 함수는 Supabase 클라이언트를 인자로 받는다. 서버(첫 페이지)와 브라우저(다음 페이지)가 같은 함수를 쓴다.

## 2. 페이지 · 라우트 목록

| 경로 | 화면 | 접근 | 하단 | 조회 | 변경 (Server Action) |
| --- | --- | --- | --- | --- | --- |
| `/` | S-01 홈 | 전체 | 탭 | 글 20개 + 작성자 + 좋아요·댓글 수, 커서 방식 무한 스크롤 | — |
| `/posts/[id]` | S-02 글 상세 | 전체 | 댓글 입력 바 | 글, 댓글 목록(작성순), 내가 좋아요했는지 | `toggleLike` · `createComment` |
| `/posts/new` | S-03 글 작성 | 회원 | — | — | `createPost` → 상세로 이동 |
| `/profile` | S-04 내정보 | 전체 | 탭 | 내 프로필, 글 수, 내가 쓴 글. 비회원이면 로그인 안내 | — |
| `/profile/edit` | S-05 프로필 수정 | 회원 | 고정 CTA | 내 프로필 | `updateProfile` (이미지는 브라우저에서 직접 업로드) |
| `/users/[id]` | S-06 유저 프로필 | 전체 | — | 프로필, 글 수, 쓴 글. 본인이면 `/profile`로 이동 | — |
| `/settings` | S-07 설정 | 전체 | 탭 | 이메일(회원), 앱 버전 | `logout` |
| `/login` | S-08 로그인 | 비회원 | 고정 CTA | — | `login` → `next` 또는 `/` |
| `/signup` | S-09 회원가입 | 비회원 | 고정 CTA | 닉네임 사용 가능 여부 | `signup` → `/` |

- 회원 전용 경로에 비회원이 오면 `/login?next=<원래 경로>`로 보낸다. 비회원 전용 경로에 회원이 오면 `/`로 보낸다.
- 좋아요·댓글처럼 화면 안에서 일어나는 동작도 비회원이면 같은 방식으로 로그인으로 보낸다.
- API 라우트는 만들지 않는다. 조회는 Supabase 직접 호출, 변경은 Server Action으로 끝난다.

## 3. DB 테이블 개요

요구사항 5장의 구조를 그대로 쓰고, 제약과 기본값을 더한다.

| 테이블 | 컬럼 | 더하는 것 |
| --- | --- | --- |
| `profiles` | id(PK, FK→auth.users) · nickname · avatar_url · created_at · updated_at | 닉네임 형식 CHECK(2~12자, 한글·영문·숫자·`_`), 대소문자 무시 유니크 인덱스, `updated_at` 트리거 |
| `posts` | id · author_id(FK→profiles) · title · content · created_at | `author_id` 기본값 `auth.uid()`, 길이 CHECK(제목 1~100, 본문 1~5000), 인덱스 `(created_at desc, id desc)` · `(author_id, created_at desc)` |
| `comments` | id · post_id(FK→posts) · author_id(FK→profiles) · content · created_at | `author_id` 기본값 `auth.uid()`, 길이 CHECK(1~500), 인덱스 `(post_id, created_at)` |
| `likes` | post_id · user_id · created_at, PK `(post_id, user_id)` | `user_id` 기본값 `auth.uid()` |

- **가입 트리거**: `auth.users`에 행이 생기면 `handle_new_user()`가 가입 때 넘긴 닉네임으로 `profiles` 행을 만든다.
- **집계**: 좋아요 수·댓글 수·글 수는 컬럼으로 두지 않고 조회할 때 센다 (`likes(count)`, `comments(count)`).
- **스토리지**: `avatars` 버킷. 공개 읽기, 5MB 제한, JPG·PNG·WebP만 허용. 경로는 `{user_id}/파일명`.
- **RLS**: 요구사항 5.5 표 그대로다.

| 테이블 | select | insert | update | delete |
| --- | --- | --- | --- | --- |
| profiles | 전체 | 정책 없음 (트리거만) | 본인 행, nickname·avatar_url 컬럼만 | 정책 없음 |
| posts | 전체 | 로그인 + `author_id = auth.uid()` | 정책 없음 | 정책 없음 |
| comments | 전체 | 로그인 + `author_id = auth.uid()` | 정책 없음 | 정책 없음 |
| likes | 전체 | 로그인 + `user_id = auth.uid()` | 정책 없음 | 본인 행 |
| storage `avatars` | 전체 | 본인 폴더 | 본인 폴더 | 본인 폴더 |

## 4. 구현 순서

각 단계는 "마이그레이션 → 조회·액션 → 화면" 순으로 진행하고, 끝날 때 완료 기준을 확인한 뒤 커밋한다.

### 0단계 — 준비

사용자가 할 일
- Supabase 프로젝트 생성 후 URL과 publishable key 전달
- Supabase 대시보드 Auth 설정에서 **Confirm email 끄기**
- `docs/design-analysis` 브랜치를 `main`에 병합할지 결정 (지금은 `main`에 `design.md`가 없다)

작업
1. `create-next-app`으로 프로젝트 생성, 의존성 설치, `.env.example` 작성
2. Supabase 클라이언트 3종(`client` · `server` · `proxy`)과 `src/proxy.ts`
3. `globals.css`에 토큰: 주색 `#FF6B57`, 회색 6단계, 빨강, 구분선 `#EBEBEB`, 모서리 4·6·8
4. 공통 레이아웃(최대 폭 480)과 UI 컴포넌트: Button, TextField, Header, BottomTab, FixedBottomCTA
5. 빈 화면 9개와 라우트 그룹 배치

완료 기준: 탭 3개가 이동되고, 탭 화면에만 하단 탭이 보인다. `npm run build`가 통과한다.

### 1단계 — 인증 (AUTH-01~03, SET-01)

1. `0001_profiles.sql` 적용, 타입 생성
2. 회원가입: 닉네임 중복 확인 → `signUp`(닉네임을 메타데이터로 전달) → 자동 로그인 → `/`
3. 로그인: `signInWithPassword` → `next` 또는 `/`
4. 접근 제어: `proxy.ts`의 리다이렉트 + `requireUser()`
5. 설정 화면: 이메일, 앱 버전, 로그아웃(확인 창)

완료 기준: 가입 → 자동 로그인 → 브라우저 재시작 후에도 유지 → 로그아웃. 중복 이메일·중복 닉네임·틀린 비밀번호에 정해진 문구가 뜬다.

### 2단계 — 피드 (POST-01~03)

1. `0002_posts.sql` 적용
2. 글 작성: 입력 검증, 등록 중 버튼 비활성, 뒤로가기 확인 창
3. 홈: PostCard, 무한 스크롤, 스켈레톤·빈 상태·오류 상태, 글쓰기 플로팅 버튼
4. 글 상세: 본문(줄바꿈 유지), 작성자, 없는 글 안내

완료 기준: 글을 쓰면 상세로 이동하고 홈 맨 위에 보인다. 21개 이상일 때 스크롤로 다음 묶음이 붙는다.

### 3단계 — 댓글 · 좋아요 (CMT-01~02, LIKE-01)

1. `0003_comments_likes.sql` 적용
2. 댓글 목록과 하단 고정 입력 바. 비회원에게는 "로그인하고 댓글 쓰기"
3. 좋아요 버튼: 누르는 즉시 반영하고 실패하면 되돌린다
4. 피드 카드에 좋아요 수·댓글 수 표시

완료 기준: 댓글이 목록 맨 아래에 붙고 수가 바뀐다. 좋아요를 두 번 누르면 원래대로 돌아오고, 새로고침해도 상태가 유지된다.

### 4단계 — 프로필 (PROF-01~03)

1. `0004_storage_avatars.sql` 적용
2. 내정보: 프로필, 글 수, 내가 쓴 글(PostList 재사용), 비회원 안내
3. 다른 유저 프로필: 본인이면 `/profile`로, 없는 유저면 안내
4. 프로필 수정: 닉네임, 이미지 선택·미리보기·삭제, 저장 후 전체 반영

완료 기준: 닉네임과 이미지를 바꾸면 피드·상세·댓글에 모두 반영된다. 5MB 초과나 다른 형식은 거절된다.

### 5단계 — 마무리

- 상세에서 뒤로 왔을 때 피드의 스크롤 위치와 불러온 목록 복원
- 현재 탭을 다시 누르면 맨 위로
- 누르는 영역 44px, 하단 안전 영역, 480px 초과 화면 확인
- 모든 요청의 처리 중·성공·실패 표시 점검 (요구사항 2.7)

### 6단계 — 배포

1. Vercel에 저장소 연결, 환경변수 2개 등록
2. Supabase Auth의 Site URL을 배포 주소로 설정
3. 배포 주소에서 요구사항 3장의 흐름 6개 확인

0단계 직후에 Vercel을 미리 연결해 두면 단계마다 미리보기 주소로 휴대폰에서 확인할 수 있다.

## 5. 주의점

### RLS · 접근 권한

- **테이블을 만드는 마이그레이션에서 RLS 켜기와 정책을 함께 넣는다.** RLS가 꺼진 테이블은 공개 키만으로 누구나 읽고 쓸 수 있다.
- **작성자는 DB가 정한다.** `author_id` · `user_id`는 기본값 `auth.uid()`와 `with check`로 고정하고, Server Action은 클라이언트가 보낸 작성자 값을 받지 않는다.
- **수정·삭제는 정책을 만들지 않는 것으로 막는다.** 정책이 없으면 거부된다.
- **profiles는 컬럼 단위로 제한한다.** 본인 행이어도 `id` · `created_at`은 바꿀 수 없게 `nickname` · `avatar_url`에만 update 권한을 준다.
- **가입 트리거는 `security definer` + `set search_path = ''`로 만든다.** 닉네임은 클라이언트가 보낸 값이므로 CHECK 제약으로 다시 검증한다.
- **스토리지 제한은 버킷에 건다.** 용량·형식을 화면에서만 검사하면 우회된다.
- **`service_role` 키는 쓰지 않는다.** 이 앱에 필요 없고, `NEXT_PUBLIC_`으로 노출하면 RLS가 전부 무력화된다.
- **단계마다 RLS를 직접 확인한다.** 비회원으로 insert, 다른 회원 명의로 insert, 남의 좋아요 삭제, 남의 프로필 수정이 모두 거부되는지 본다.

### 로그인 · 세션

- **서버에서는 `getUser()`(또는 `getClaims()`)로 확인한다.** `getSession()`은 쿠키를 검증 없이 읽으므로 권한 판단에 쓰지 않는다.
- **`proxy.ts`가 요청마다 세션 쿠키를 갱신한다.** 서버 컴포넌트는 쿠키를 쓸 수 없어서 이것이 빠지면 로그인이 중간에 풀린다.
- **접근 제어는 세 겹이다.** proxy의 리다이렉트는 편의, 페이지와 Server Action의 확인은 방어, 실제 차단은 RLS가 한다.
- **`next` 값은 검증한다.** `/`로 시작하고 `//`로 시작하지 않는 내부 경로만 허용한다. 그대로 쓰면 외부 사이트로 보내는 데 악용된다.
- **Confirm email이 켜져 있으면 요구사항대로 동작하지 않는다.** 가입 직후 자동 로그인이 안 되고, 중복 이메일도 오류로 돌아오지 않는다.
- **닉네임 중복은 가입 전에 미리 확인한다.** 트리거에서 실패하면 Supabase가 원인을 알 수 없는 오류로 돌려준다. 동시에 가입하는 경우는 유니크 인덱스가 막고, 그 오류도 닉네임 문구로 바꿔 보여준다.
- **로그인 실패 문구는 하나로 통일한다.** 이메일과 비밀번호 중 어느 쪽이 틀렸는지 드러내지 않는다.
- **로그인 상태에 따라 달라지는 화면은 캐시하지 않는다.** 로그아웃 후에는 `revalidatePath('/', 'layout')`로 남은 화면을 비운다.

### 그 밖

- **프로필 이미지는 브라우저에서 스토리지로 직접 올린다.** Server Action은 기본 1MB, Vercel 함수는 4.5MB에서 요청이 잘려 5MB 파일이 통과하지 못한다.
- **이미지를 바꿀 때마다 파일명을 새로 만든다.** 같은 이름으로 덮어쓰면 캐시 때문에 옛 이미지가 계속 보인다.
- **본문과 댓글은 텍스트로만 그린다.** `whitespace-pre-wrap`으로 줄바꿈을 살리고 HTML로 넣지 않는다.
- **무한 스크롤은 커서 방식으로 한다.** `(created_at, id)` 기준으로 이어 받아야 새 글이 올라와도 중복이 생기지 않는다.
- **입력창 글자는 16px로 한다.** 디자인은 약 14px이지만 iOS Safari는 16px 미만 입력창에 포커스하면 화면을 확대한다.
- **Next.js 버전별 차이를 설치 후 확인한다.** `params` · `cookies()`가 비동기인지, 파일명이 `proxy.ts`인지 `middleware.ts`인지가 버전에 따라 다르다.

## 검증

- **매 단계**: `npm run lint`, `npm run build`, 해당 단계의 완료 기준을 브라우저 모바일 폭(390px)에서 확인
- **RLS**: Supabase SQL 편집기 또는 공개 키만 쓴 스크립트로 "비회원 / 회원 A / 회원 B" 세 입장에서 허용·거부를 확인
- **전체 흐름**: 요구사항 3.1~3.6의 여섯 흐름을 계정 두 개로 처음부터 끝까지 수행
- **배포 후**: 실제 휴대폰(iOS Safari, Android Chrome)에서 하단 안전 영역, 키보드가 올라왔을 때 댓글 입력 바, 로그인 유지를 확인
