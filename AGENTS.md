# AGENTS.md - Developer Guidelines for AI Agents (Loomer)

Welcome, AI Agent! You are working on **Loomer**, a high-performance, privacy-first, minimalist desktop and mobile browser built with **Tauri v2 (Rust Core)** and **Angular (Zoneless + Signals)**.

Loomer is a clean-sheet multiplatform rewrite and evolution of the open-source `firn-legacy` browser under the **Mozilla Public License 2.0 (MPL-2.0)**.

This document defines the architectural rules, legal/branding requirements, coding standards, and workflow protocols you MUST follow when executing tasks from `TASKS.md`.

---

## 1. Branding & MPL-2.0 Compliance Rules (CRITICAL)

Per the original author's notice in `firn-legacy`:
1. **No "Firn" Branding or Icons**: The product name is **Loomer** (`loomer`). Never use the name "Firn" in user-facing UI text, window titles, package names, or bundle identifiers, and **DO NOT** copy or bundle `legacy/assets/icon.*`. Use Loomer's own icons in `./assets/`.
2. **Exclude Farewell Logic**: Do **NOT** port `legacy/src/farewell.ts`, `FAREWELL_URL`, `farewellSeen`, or the `farewell` overlay mode. That logic existed solely to announce the archival of the legacy Electron repository.
3. **MPL-2.0 Attribution**: Keep `license = "MPL-2.0"` across all `Cargo.toml` and `package.json` files. When porting or adapting files directly from `./legacy/` (such as `styles.css` or `page-preload.ts`), include an attribution header at the top of the file:
   `// This Source Code Form is subject to the terms of the Mozilla Public License, v. 2.0.`
   `// Original work Copyright (c) Firn / David. Modified work Copyright (c) Loomer Contributors.`

---

## 2. Core Architecture & Boundaries

1. **Monorepo Structure**:
   - `crates/loomer-core/`: 100% platform-agnostic Rust business logic (`TabManager`, `Spaces`, `History`, `AdBlocker`, `Engine` traits). **NO Tauri or GUI dependencies here.**
   - `crates/loomer-tauri/`: Tauri v2 host application, multi-webview compositor, and platform-specific engine adapters (`WryDesktopEngine`, mobile plugins).
   - `packages/core-ui/`: (`@loomer/core-ui`) Shared Angular library containing types, transport interfaces (`LoomerTransport`), and reactive state management (`LoomerBridgeService` using Angular Signals).
   - `packages/desktop-ui/`: (`@loomer/desktop-ui`) Desktop frontend application organized into a 2-layer architecture (`ui-shell` + `ui-overlay`).
   - `packages/mobile-ui/`: (`@loomer/mobile-ui`) Mobile fork package (Angular + Ionic).

2. **The 2-Layer UI Compositor (Desktop)**:
   - **Layer 1 (`ui-shell`)**: Full-window transparent background (`#00000000`), renders the docked sidebar and top bar.
   - **Layer 2 (`ui-overlay`)**: Transparent overlay webview positioned above web pages, activated only when Command Bar, Peek sidebar, Find, Lookout, or permission prompts are open.
   - **Tab Viewports**: Managed separately by the Rust compositor as child webviews (`tab-*`) positioned inside the coordinate space `(pageLeft, pageTop)` to avoid overlapping the shell.

3. **Strict Separation of Concerns**:
   - Angular components **MUST NOT** import Tauri APIs directly. All communication happens via the `LOOMER_TRANSPORT` injection token and `LoomerBridgeService`.
   - Rust code in `loomer-core` **MUST NOT** depend on Tauri. It communicates through the `BrowserEngine` and `EnginePage` traits.

---

## 3. Coding Standards & Conventions

### Rust (`loomer-core` & `loomer-tauri`)
- **Error Handling**: Use `thiserror` for descriptive domain errors. Avoid `.unwrap()` or `.expect()` in production code paths unless the invariant is mathematically guaranteed and documented.
- **Async Runtime**: Use `tokio` for asynchronous tasks (file I/O, network requests, Safe Browsing checks).
- **Serialization**: Ensure `serde` structs match the exact JSON schema of legacy files (`session.json`, `settings.json`, `history.json`, etc.) to guarantee 100% backward compatibility.
- **Performance**: Keep serialization overhead minimal. Use `adblock-rust` for synchronous request filtering.

### TypeScript / Angular (`core-ui` & `desktop-ui`)
- **Zoneless & Signals**: The app runs with `provideExperimentalZonelessChangeDetection()` (or `provideZonelessChangeDetection()` in Angular 20+). Do **NOT** use `Zone.js` or `setTimeout` wrappers for change detection. Use Angular Signals (`signal`, `computed`, `effect`) for all reactive state.
- **Component Prefix**: Use `loomer-` selector prefixes (e.g., `loomer-root`, `loomer-sidebar`).
- **OnPush Change Detection**: Every component must use `changeDetection: ChangeDetectionStrategy.OnPush`.
- **Styling**: Respect CSS custom properties and design tokens from `legacy/src/ui/styles.css`.

---

## 4. Legacy Codebase Reference Map (`./legacy/`)

The original Electron + React codebase is located in `./legacy/` for reference, schema compatibility, and M1-T4 harness testing.
- **DO NOT** modify files in `./legacy/` (except temporarily pointing the UI URL on a local test branch for M1-T4 testing).
- **ALWAYS** inspect the corresponding `./legacy/` file before implementing a task in `loomer-core` or `packages/*` to preserve 100% functional and schema parity:

| Target in `loomer` | Reference in `./legacy/` | What to extract / port |
| :--- | :--- | :--- |
| `packages/core-ui/src/models/` | `legacy/src/types.ts` | Exact TypeScript interfaces (`CarryTab`, `FrameState`, `OverlayState`, `SavedWindow`, `Space`) — omit `farewell` mode. |
| `packages/desktop-ui/src/styles/` | `legacy/src/ui/styles.css` | CSS custom properties (`--motion`, `--top-bar-height`), glass tints, and layout styles. |
| `crates/loomer-core/src/engine.rs` | `legacy/src/engine/engine.ts` | Exact method signatures and callbacks for `Page` and `Engine` traits (M2-T1). |
| `crates/loomer-core/src/tabs.rs` | `legacy/src/tabs.ts` & `archive.ts` | `TabManager`, `BASECAMP_MAX`, split view math, `sleepOld`, and `archiveOld` logic (M2-T2). |
| `crates/loomer-core/src/storage.rs` | `legacy/src/store.ts` & `settings.ts` | Exact JSON serialization keys for `session.json` and `settings.json` + `SaveScheduler` (M2-T3). |
| `crates/loomer-core/src/adblock.rs` | `legacy/src/adblock.ts` | Filter list initialization, binary caching, and `adsAllowedSites` domain matching (M2-T4). |
| `crates/loomer-core/src/safebrowsing.rs` | `legacy/src/safebrowsing.ts` | Google Safe Browsing hash prefix check and threat types (M2-T4). |
| `crates/loomer-tauri/src/compositor.rs` | `legacy/src/main.ts` | Constants (`SIDEBAR_WIDTH`, `PEEK_*`, `TOP_BAR_*`), `boundsFor`, `glidePageLeft`, and `watchEdges` (M3-T3). |
| `crates/loomer-tauri/assets/page-preload.js` | `legacy/src/page-preload.ts` | Login form detection and Shift+Click Lookout interception script (M3-T2). |

---

## 5. Agent Execution Protocol

When assigned a task from `TASKS.md`:

1. **Read & Analyze**: Read the task description, acceptance criteria, and corresponding `./legacy/` reference files thoroughly.
2. **Atomic Implementation**: Implement changes cleanly in the appropriate workspace package/crate.
3. **Verify Tests & Types**:
   - Rust: Run `cargo test` to verify backend logic.
   - TypeScript: Run `pnpm typecheck` to ensure zero type errors.
4. **Update Tracking**: Once complete and verified, mark the task as checked (`- [x]`) in `TASKS.md`.
