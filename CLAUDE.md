# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **1단계 시연 (Phase-1 demo / wireframe)** for (주)배움's 학습설계 (learning-design) system — a proposal to replace an Excel-spreadsheet workflow with structured-data screens. All numbers and copy are illustrative placeholders meant to convey flow and direction, not production logic. The UI is entirely in Korean.

## Build / run / test

There is **no build system, package manager, dependencies, or tests**. The app is four self-contained static HTML files. To work on it, open the files directly in a browser (`file://`) or serve the folder statically, e.g.:

```
python -m http.server 8000   # then visit http://localhost:8000/index.html
```

Each file inlines all CSS (`<style>`) and JS (`<script>`) — there are no external assets, fonts loaded by URL, or shared libraries. Editing means editing one HTML file in place.

## Architecture

Four pages, linked only by `<a href>`/`location.href` navigation — there is **no shared state, backend, or persistence** (no localStorage/fetch/API). Each page seeds its own hardcoded demo data as JS object literals at the top of its script and re-renders from those in-memory arrays. Reloading resets everything.

- **`index.html`** — demo home; hub linking to the four screens. Describes the intended demo flow (learner applies → counselor receives → counselor builds & sends → student views).
- **`builder.html`** (~1300 lines, the core) — the counselor's design tool **and** the student-facing result view, in one file.
- **`admin.html`** — counselor's auxiliary tools: design list (received→expired lifecycle), templates, subject sets, copy/message settings. Tab-switched via `switchMenu()`.
- **`learner.html`** — learner LMS "my page": application form + cumulative list of requests with status flow. Toggles list/form via `switchV()` and phone/PC via `switchDev()`.

Every page shares the same top nav (GNB) markup and the CSS custom-property palette (`--main:#0093D0`, etc.) copied into each file.

### builder.html — key concepts

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
