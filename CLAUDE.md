# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **1단계 시연 (Phase-1 demo / wireframe)** hub for (주)배움's system redesign proposals. It currently hosts two independent demo projects, each proposing to replace part of the current Excel/legacy workflow with structured-data screens:

- **`learning-plan/`** - 학습설계 (learning-design): counselor-built learning plans, replacing an Excel-spreadsheet workflow with structured-data screens.
- **`enrollment/`** - 수강신청 (course-enrollment) redesign: a full-frame wireframe for the course selection/registration flow (과정 → 방식 → 개강일 선택, 패키지/단과 신청, 장바구니, 결제 확인).

All numbers and copy across both are illustrative placeholders meant to convey flow and direction, not production logic. The UI is entirely in Korean.

## Build / run / test

There is **no build system, package manager, dependencies, or tests**. The app is eight self-contained static HTML pages (seven demo screens + one shared root hub) plus two shared assets (`assets/tokens.css`, `assets/packages.js`). To work on it, open the files directly in a browser (`file://`) or serve the folder statically, e.g.:

```
python -m http.server 8000   # then visit http://localhost:8000/index.html
```

Each file inlines almost all CSS (`<style>`) and JS (`<script>`); the exceptions are the two shared assets - `assets/tokens.css` (brand colors, linked by every page) and `assets/packages.js` (the package seed, loaded by `proposal/` and `enrollment/`). There are no fonts loaded by URL or other shared libraries. Editing means editing one HTML file in place (plus a shared asset if a brand color or a package value itself needs to change).

## Architecture

The repo is a monorepo hub for multiple independent demo projects, plus one shared CSS file:

- **`index.html`** (root) - top-level hub linking to each project below. No shared state with either project; purely a set of `<a href>` cards.
- **`assets/tokens.css`** - CSS custom properties for the three orgs' brand colors (배움/배론/허브, each with `main`/`mid`/`soft`, e.g. `--baeron-main:#0093D0`). Every page links it via `<link rel="stylesheet">` in `<head>`, then maps its own local `--main`/`--mid`/`--soft` variables (and, in `builder.html`/`admin.html`, the `--blue`/`--blue-soft`/`--blue-line` aliases) to the matching `--baeron-*` token instead of hardcoding hex - this is the one color source of truth across all pages.
- **`assets/packages.js`** - the package seed (SSOT): the 29 products of the LMS '패키지관리' list as one set, collected 2026.08.20 (차수/모집기간/진행상태/개강반/구성과목) and 2026.08.26 (분류/패키지명/과목수/판매가). `proposal/` reads the schedule axis via `pkgSlots()`/`pkgRounds()`/`PKG_COMPO`; `enrollment/` reads the price axis via `pkgAdminRows()`. Product names follow the admin list so a learner sees the same name in a proposal and at checkout. Globals are `PKG_`-prefixed to avoid colliding with page-local names (`enrollment/` has its own `PKG_SUBJ`). Known drift, deliberately left visible: 한국어교원's 목록 과목수 (16 · 15) differs from 상세 구성 (14 · 13) - both values are kept and the mismatch is surfaced in the proposal popup.
- **`docs/`** - project documentation per `## 표준` below (학습설계툴 논의이력, 패키지화면 개선기획 계획).

### `learning-plan/` - 학습설계 (learning-design) demo

Four pages, linked only by `<a href>`/`location.href` navigation among themselves - there is **no shared state, backend, or persistence** (no localStorage/fetch/API). Each page seeds its own hardcoded demo data as JS object literals at the top of its script and re-renders from those in-memory arrays. Reloading resets everything.

- **`index.html`** - sub-hub for this project; links to the four screens below. Describes the intended demo flow (learner applies → counselor receives → counselor builds & sends → student views).
- **`builder.html`** (~1300 lines, the core) - the counselor's design tool **and** the student-facing result view, in one file.
- **`admin.html`** - counselor's auxiliary tools: design list (received→expired lifecycle), templates, subject sets, copy/message settings. Tab-switched via `switchMenu()`.
- **`learner.html`** - learner LMS "my page": application form + cumulative list of requests with status flow. Toggles list/form via `switchV()` and phone/PC via `switchDev()`.

Every page in this folder shares the same top nav (GNB) markup, copied into each file; their internal links (`index.html`, `builder.html`, etc.) are relative to this folder and don't reach the root hub.

#### builder.html - key concepts

State lives in top-level `let` variables; UI is rebuilt by calling `render*()` functions after each mutation. Central toggles:

- **`mode`** - `"cert"` (자격증 / social-worker certificate) vs `"deg"` (학위 / degree). Nearly every function branches on `mode`. `sems()`/`ihList()` return the active dataset (`certSems`/`degSems`, `certIH`/`degIH`). `switchMode()` swaps it.
- **Two screens in one file**, switched by `switchView('admin'|'student')`: `#adminView` (the builder) and `#studentView` (the sent result). `?view=student` in the URL jumps straight to the student view on load (this is how `learner.html`, `admin.html`, and `index.html` deep-link into it).
- **`device`** - `"mo"` vs `"pc"`; the student view renders differently (`renderStudent()` vs `renderStudentPC()`).
- **`builderLayout`** (`stack`/`tab`/`split`) - alternate wireframe layouts for the builder, an intentional A/B of the design.

Core domain model:
- **`ihList()`** = 기이수 현황 (prior coursework). Each item: `{id, name, credit, area, status, kind, origin, when}`. `status` ∈ `recog`/`excl`/`unset` (인정/제외/미분류); `kind` ∈ `sub`/`cr`/`pre` (subject / credit-based cert / pending-cert); `origin` ∈ `internal`/`transfer`/`cert`. `area` classification differs by mode: cert uses `major`/`gen`/`free`; degree uses `mreq`/`melec`/`gen`/`free` (전필/전선/교양/일선), labeled via `AREA_META[mode]`.
- **`sems()`** = planned semesters, each `{subjects:[...]}`. Terms are labeled from `START_YEAR`/`START_TERM` via `termLabel()`.
- Pricing: `PRICE_PER` per subject, `QTY_DISCOUNT` volume tiers, `PKG_FULL` package price (cert only). Validation constants: `SEM_MAX` (per-semester credit cap), `CERT_CREDITS`.

When adding features, follow the existing pattern: mutate the in-memory arrays, then call the relevant `render*()`; keep `mode`-awareness in any new domain logic.

### `enrollment/` - 수강신청 (course-enrollment) redesign demo

- **`index.html`** - a single self-contained page (no internal navigation, no links to other files). Same static/no-backend, hardcoded-demo-data model as `learning-plan/`. Package rows come from `assets/packages.js` (`pkgAdminRows()`); everything the admin list does not carry - 구성 과목 · 정가 · 할인율 - is filled from this page's own subject data as an example.

### `proposal/` - 원클릭 제안 (one-click proposal) demo

**`index.html`** - 패키지 기반 제안형. Two screens in one file, switched by `switchView('admin'|'student')`; the learner view also toggles 모바일/PC via `switchDevice()`.

- **Who it targets** - learners who have **not** applied for 학습설계. Applying is the learner's own action and applications go to `learning-plan/`'s builder for a hand-built design; this screen pushes a proposal to people who never apply. That is why the list carries no 신청구분/실명인증 column.
- **담당자 화면 = 제안 목록 관리** - columns `No · 담당 · 이름(ID) · 연락처 · 최종학력 · 목표과정 · 제안하기 · 상태 · 열람 여부 · 결제 여부`. `내 담당`(default) / `팀 전체` scope via `scopeOwn`, `미제안만` filter via `onlyNew`, and a 현행-LMS-style summary (`총 N개 · p/전체 페이지 · 한 화면에`) via `perPage`/`page`.
- **Rows are proposal records, not people** (`listRows()`) - re-proposing does not overwrite; a new row stacks and the same learner appears more than once (`N차` badge, past rows dimmed). 열람·결제 belong to that round. `No` is a per-record 제안번호 (`pno`, `nextPno()`); an unsent row has none.
- **Domain model** - each learner in `APPS` carries `props`, an array of send rounds `{pno, sentDt, view, pay, picks, cmt, memo}`. `rounds()`/`lastRound()`/`stOf()`/`roundPicks()` read it. `cmt` is learner-facing (shown on the student screen); `memo` is internal-only.
- **제안안 popup** - `oneClick()` builds it from rules (`autoPicks()` = 학력 매핑 `RULE` + 주소지 `regionOf()` + 슬롯별 모집중 최이른 개강 `pickPkg()`); `viewSent(id, seq)` reopens a sent round read-only (`propRO`, `curSeq`) with 회차 탭; `newRound()` starts a re-proposal. **Re-proposal re-runs the rules rather than copying the snapshot** - copying would resend a closed 차수 - and `diffLine()` shows what changed against the previous send.
- **수강료는 이 화면에 표시하지 않는다** (2026.08.20 결정). 유효기간 is derived from the selected 개강반's 모집 마감일 (`expireOf()`); the demo clock is fixed at 2026-08-20 in `ddayText()`.

### `packages/` - 패키지 화면 개선 demo

- **`index.html`** - career.baeoom.com 랜딩의 "배움 하나로 패키지" 섹션 개선안. Its `CUR_PKGS`/`P` values are the **랜딩 실측 (2026-08-13, 정가·최대할인가)** - a different snapshot from the admin 판매가 in `assets/packages.js`, kept separate on purpose since this page's argument is a diagnosis of the current screen.

## 표준

DB·데이터·문서 관리는 https://github.com/juhuikim-baeoom/web-standards 채택.

- 네이밍: snake_case, 예약어 회피, 표준 약어(TP/CD/NM/UUID/AT/FLAG/SEQ)
- 테이블: 목적 기반 접두사, org_cd 다기관 구분, 감사 컬럼, soft-delete
- 용어: 신규 엔티티·필드명은 00-domain-glossary.md 우선 조회, 없으면 등록 후 사용
- 문서: SSOT, 코드 변경 시 문서 동기화, Mermaid 사용
- 문서 동기화는 04-document-management-rules.md를 따른다
