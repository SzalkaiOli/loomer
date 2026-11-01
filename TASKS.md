# TASKS.md - Implementation Roadmap & Task Breakdown (Loomer)

This file tracks the development milestones and granular tasks for implementing **Loomer**. AI agents must check off tasks (`[x]`) upon successful completion and testing.

---

## Milestone 1: Monorepo Setup & Angular UI (@loomer/core-ui + desktop-ui)
- [ ] **M1-T1**: Initialize pnpm workspace, Cargo workspace structure, and custom Loomer placeholder icons in `./assets/`.
  - *Acceptance*: Root `Cargo.toml`, `pnpm-workspace.yaml`, and `tsconfig.base.json` configured correctly; `cargo check` and `pnpm install` succeed.
- [ ] **M1-T2**: Implement `@loomer/core-ui` library with `LoomerTransport`, models (`TabSummary`, `Space`, `NavState`, `OverlayState` — excluding legacy `farewell` mode), and `LoomerBridgeService` (Angular Signals).
  - *Acceptance*: Library builds without errors and provides reactive state wrappers.
- [ ] **M1-T3**: Build 2-layer Angular desktop application structure in `desktop-ui` (`ui-shell` for Sidebar/TopBar, `ui-overlay` for floating layers) using `loomer-` component prefixes.
  - *Acceptance*: App correctly handles query parameter `?view=overlay` to render the correct layer.
- [ ] **M1-T4**: Implement temporary `ElectronLoomerTransport` adapter for backwards-compatibility testing against legacy Electron backend.
  - *Acceptance*: Angular UI renders successfully inside the legacy Electron harness.

---

## Milestone 2: Rust Core Library (`loomer-core`)
- [ ] **M2-T1**: Define platform-agnostic `BrowserEngine` and `EnginePage` traits in `loomer-core/src/engine.rs`.
  - *Acceptance*: Traits compile cleanly with zero GUI/Tauri dependencies.
- [ ] **M2-T2**: Port `TabManager`, SplitView, Basecamp, Tab Sleep (`sleepOld`), and Archiving (`archiveOld`) logic to `loomer-core/src/tabs.rs`.
  - *Acceptance*: Full test coverage for tab lifecycle and split positioning.
- [ ] **M2-T3**: Implement JSON storage modules (`session.json`, `settings.json`, `history.json`, `permissions.json`, `passwords.json`) using `serde` and `SaveScheduler`.
  - *Acceptance*: Can read and write valid JSON state files compatible with the legacy format.
- [ ] **M2-T4**: Integrate `adblock-rust` and SafeBrowsing local hash verification modules.
  - *Acceptance*: Ad blocking and threat detection rules evaluate correctly in unit tests.

---

## Milestone 3: Tauri v2 Host & Wry Desktop Engine (`loomer-tauri`)
- [ ] **M3-T1**: Configure `tauri.conf.json` (`com.loomerbrowser.loomer`) with transparent frame settings, security CSP, and default Tauri capabilities.
  - *Acceptance*: Tauri app window launches with frosted glass / acrylic styling under the "Loomer" title.
- [ ] **M3-T2**: Implement `WryDesktopEngine` using Tauri v2 multi-webview child management (`window.add_child`) and `page-preload.js` script injection.
  - *Acceptance*: Child webviews load external URLs and execute preload scripts correctly.
- [ ] **M3-T3**: Implement 2-layer UI compositor and native Edge-Watcher timer in Rust (`compositor.rs`).
  - *Acceptance*: Mouse cursor touching screen edges triggers peek sidebar and top bar glide animations.
- [ ] **M3-T4**: Implement `TauriLoomerTransport` in `@loomer/core-ui` and connect Tauri IPC commands (`#[tauri::command]`).
  - *Acceptance*: End-to-end communication between Angular frontend and Rust backend works seamlessly.

---

## Milestone 4: Native OS Integrations & Security
- [ ] **M4-T1**: Integrate `keyring` crate into `PasswordStore` for secure OS-backed credential storage.
  - *Acceptance*: Passwords are encrypted/decrypted via system keychain/credential manager.
- [ ] **M4-T2**: Implement `OwnerGate` biometrics / system authentication wrapper for revealing passwords.
  - *Acceptance*: Prompting for password reveal triggers OS biometric/pin authentication where supported.
- [ ] **M4-T3**: Setup single-instance lock and CLI command-line argument link handlers (`second-instance`, `open-url`, `open-file`).
  - *Acceptance*: Opening external links focuses the existing Loomer instance and opens a new tab.

---

## Milestone 5: Mobile Fork Preparation (Optional)
- [ ] **M5-T1**: Create `packages/mobile-ui` (`@loomer/mobile-ui`) package using Angular + Ionic Standalone components sharing `@loomer/core-ui`.
  - *Acceptance*: Mobile shell renders bottom bar and bottom-sheet modals correctly.
- [ ] **M5-T2**: Implement Tauri Mobile plugin engine adapter (`engine_mobile.rs`) for Android WebView and iOS WKWebView.
  - *Acceptance*: Rust core successfully drives native mobile webviews through abstract engine traits.
