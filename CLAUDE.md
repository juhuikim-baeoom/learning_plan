# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **1단계 시연 (Phase-1 demo / wireframe)** hub for (주)배움's system redesign proposals. It currently hosts two independent demo projects, each proposing to replace part of the current Excel/legacy workflow with structured-data screens:

- **`learning-plan/`** — 학습설계 (learning-design): counselor-built learning plans, replacing an Excel-spreadsheet workflow with structured-data screens.
- **`enrollment/`** — 수강신청 (course-enrollment) redesign: a full-frame wireframe for the course selection/registration flow (과정 → 방식 → 개강일 선택, 패키지/단과 신청, 장바구니, 결제 확인).

All numbers and copy across both are illustrative placeholders meant to convey flow and direction, not production logic. The UI is entirely in Korean.

## Build / run / test

There is **no build system, package manager, dependencies, or tests**. The app is six self-contained static HTML files (five demo screens + one shared root hub) plus one shared CSS file (`assets/tokens.css`). To work on it, open the files directly in a browser (`file://`) or serve the folder statically, e.g.:

```
python -m http.server 8000   # then visit http://localhost:8000/index.html
```

Each file inlines almost all CSS (`<style>`) and JS (`<script>`) — the one exception is `assets/tokens.css`, linked by every page for shared brand-color variables (see Architecture below). There are no fonts loaded by URL or other shared libraries. Editing means editing one HTML file in place (plus `tokens.css` if a brand-color value itself needs to change).

## Architecture

The repo is a monorepo hub for multiple independent demo projects, plus one shared CSS file:

- **`index.html`** (root) — top-level hub linking to each project below. No shared state with either project; purely a set of `<a href>` cards.
- **`assets/tokens.css`** — CSS custom properties for the three orgs' brand colors (배움/배론/허브, each with `main`/`mid`/`soft`, e.g. `--baeron-main:#0093D0`). Every page links it via `<link rel="stylesheet">` in `<head>`, then maps its own local `--main`/`--mid`/`--soft` variables (and, in `builder.html`/`admin.html`, the `--blue`/`--blue-soft`/`--blue-line` aliases) to the matching `--baeron-*` token instead of hardcoding hex — this is the one color source of truth across all pages.
- **`docs/`** — currently empty (placeholder `.gitkeep`); reserved for project documentation per `## 표준` below.

### `learning-plan/` — 학습설계 (learning-design) demo

Four pages, linked only by `<a href>`/`location.href` navigation among themselves — there is **no shared state, backend, or persistence** (no localStorage/fetch/API). Each page seeds its own hardcoded demo data as JS object literals at the top of its script and re-renders from those in-memory arrays. Reloading resets everything.

- **`index.html`** — sub-hub for this project; links to the four screens below. Describes the intended demo flow (learner applies → counselor receives → counselor builds & sends → student views).
- **`builder.html`** (~1300 lines, the core) — the counselor's design tool **and** the student-facing result view, in one file.
- **`admin.html`** — counselor's auxiliary tools: design list (received→expired lifecycle), templates, subject sets, copy/message settings. Tab-switched via `switchMenu()`.
- **`learner.html`** — learner LMS "my page": application form + cumulative list of requests with status flow. Toggles list/form via `switchV()` and phone/PC via `switchDev()`.

Every page in this folder shares the same top nav (GNB) markup, copied into each file; their internal links (`index.html`, `builder.html`, etc.) are relative to this folder and don't reach the root hub.

#### builder.html — key concepts

State lives in top-level `let` variables; UI is rebuilt by calling `render*()` functions after each mutation. Central toggles:

- **`mode`** — `"cert"` (자격증 / social-worker certificate) vs `"deg"` (학위 / degree). Nearly every function branches on `mode`. `sems()`/`ihList()` return the active dataset (`certSems`/`degSems`, `certIH`/`degIH`). `switchMode()` swaps it.
- **Two screens in one file**, switched by `switchView('admin'|'student')`: `#adminView` (the builder) and `#studentView` (the sent result). `?view=student` in the URL jumps straight to the student view on load (this is how `learner.html`, `admin.html`, and `index.html` deep-link into it).
- **`device`** — `"mo"` vs `"pc"`; the student view renders differently (`renderStudent()` vs `renderStudentPC()`).
- **`builderLayout`** (`stack`/`tab`/`split`) — alternate wireframe layouts for the builder, an intentional A/B of the design.

Core domain model:
- **`ihList()`** = 기이수 현황 (prior coursework). Each item: `{id, name, credit, area, status, kind, origin, when}`. `status` ∈ `recog`/`excl`/`unset` (인정/제외/미분류); `kind` ∈ `sub`/`cr`/`pre` (subject / credit-based cert / pending-cert); `origin` ∈ `internal`/`transfer`/`cert`. `area` classification differs by mode: cert uses `major`/`gen`/`free`; degree uses `mreq`/`melec`/`gen`/`free` (전필/전선/교양/일선), labeled via `AREA_META[mode]`.
- **`sems()`** = planned semesters, each `{subjects:[...]}`. Terms are labeled from `START_YEAR`/`START_TERM` via `termLabel()`.
- Pricing: `PRICE_PER` per subject, `QTY_DISCOUNT` volume tiers, `PKG_FULL` package price (cert only). Validation constants: `SEM_MAX` (per-semester credit cap), `CERT_CREDITS`.

When adding features, follow the existing pattern: mutate the in-memory arrays, then call the relevant `render*()`; keep `mode`-awareness in any new domain logic.

### `enrollment/` — 수강신청 (course-enrollment) redesign demo

- **`index.html`** — a single self-contained page (no internal navigation, no links to other files). Same static/no-backend, hardcoded-demo-data model as `learning-plan/`.

## 표준

DB·데이터·문서 관리는 https://github.com/juhuikim-baeoom/web-standards 채택.

- 네이밍: snake_case, 예약어 회피, 표준 약어(TP/CD/NM/UUID/AT/FLAG/SEQ)
- 테이블: 목적 기반 접두사, org_cd 다기관 구분, 감사 컬럼, soft-delete
- 용어: 신규 엔티티·필드명은 00-domain-glossary.md 우선 조회, 없으면 등록 후 사용
- 문서: SSOT, 코드 변경 시 문서 동기화, Mermaid 사용
- 문서 동기화는 04-document-management-rules.md를 따른다
