# CLAUDE.md — Commercial Mobile Game Engineering Constitution

> **BINDING DIRECTIVE:** Every rule in this file is mandatory for all sessions, sub-agents, features, refactors, and bug fixes. Never override or bypass these standards unless explicitly instructed by the user.

---

## 1. PROJECT CONFIG (UPDATE PER PROJECT)
- **Game Name:** `[PROJECT_NAME]`
- **Tech Stack:** `[e.g., Godot 4 / Unity / Flutter+Flame / React Native / Capacitor+PixiJS/Three.js]`
- **Target Platforms:** iOS (iPhone + iPad), Android (Phones + Tablets)
- **Docs Reference:** Read `docs/MARKET_AND_GDD.md` and `docs/JUICE_AND_UX_SPEC.md` before modifying gameplay or UI.
- **Verification Commands:**
  - `TYPECHECK:` `[e.g., npm run typecheck]`
  - `LINT:` `[e.g., npm run lint]`
  - `TEST:` `[e.g., npm test]`
  - `BUILD:` `[e.g., npm run build]`

---

## 2. NON-NEGOTIABLE ENGINEERING DISCIPLINE
1. **Zero Placeholders:** Never leave `TODO`, `FIXME`, stubbed handlers, dead buttons, dummy screens, or lorem ipsum in committed or "finished" code.
2. **Zero Silent Scope Cuts:** Never simplify mechanics, drop animations, remove audio/haptics, or downgrade UI polish to save effort.
3. **Strict Architectural Separation:**
   - `core/` (Engine, Rules, Math, State Machine): **Pure, deterministic, platform-agnostic, and 100% unit-testable.** Zero UI/rendering imports inside `core/`.
   - `state/` (Persistence, Save Schema, Integrity): Centralized, versioned, atomic.
   - `systems/` (Audio Manager, Haptics Manager, Input, Lifecycle): Isolated singletons/services with clean teardown.
   - `ui/` & `render/` (HUD, Screens, Animations, Particles): Purely reactive to engine state.
4. **Root-Cause Fixes Only:** Never mask bugs with empty `try/catch` blocks, loosened compiler/linter rules, skipped tests, or arbitrary `setTimeout` hacks.
5. **No Feature Deletion as a Fix:** Fixing a bug must never silently remove intended player functionality.

---

## 3. CROSS-PLATFORM & RESPONSIVE LAYOUT LAWS
All screens, HUDs, modals, and playfields MUST work cleanly across these **5 mandatory viewports**:
- **Compact Phone:** `375×667` (iPhone SE)
- **Modern iPhone Pro / Pro Max:** `393×852` & `430×932` (Dynamic Island / Notch + Bottom Home Indicator)
- **Tall Android Flagship:** `412×915` (Samsung Galaxy S/Ultra 20:9 + Gesture Bar)
- **iPad:** `768×1024` & `1024×768` (4:3)
- **Android Tablet:** `800×1280` (16:10)

**Layout enforcement:**
- **Safe Area Strictness:** Apply Safe Area insets on all edges; no HUD element, score, or button may ever overlap notches, camera cutouts, rounded corners, or system gesture bars.
- **True Tablet Adaptation:** Never stretch a phone UI to fill a tablet. Center/frame playfields cleanly and scale HUD proportions intentionally.
- **Touch Ergonomics:** Minimum interactive hit area of `44×44pt` / `48×48dp`. Prevent accidental overlap between adjacent buttons.
- **Android Back Navigation:** Hardware/gesture Back must behave deterministically on every screen (close modal -> pause active game -> exit prompt on home screen). Never freeze or trap the user.

---

## 4. GAME FEEL ("JUICE"), UX, AUDIO & HAPTICS LAWS
- **FTUE (First 5–10 Seconds):** Players must grasp the core mechanic immediately through interactive play—never walls of tutorial text.
- **Tactile Feedback:** Use spring/bounce easing curves (never robotic linear motion for UI/impacts), pressed button states, subtle impact particles, floating score/combo feedback, and snappy zero-friction retry loops.
- **Visual Clarity & Accessibility:** Strong figure-ground separation and high contrast. Never rely on color alone for gameplay-critical meaning—always pair color with distinct shape, icon, or pattern (colorblind-safe).
- **Audio System:**
  - Every key interaction must have crisp SFX; combo streaks escalate pitch/intensity; rapid SFX must be pooled/throttled to prevent clipping.
  - Independent, persisted **Music** and **SFX** toggles/volumes.
  - **Mandatory:** Immediately suspend/mute all audio when the app enters background or loses audio focus, and restore cleanly on resume.
- **Haptics System:**
  - Map distinct native haptic intensities (`Light`, `Medium`, `Heavy`, `Success`, `Error`) to real gameplay events.
  - Provide graceful no-op fallback on unsupported devices (zero runtime errors) and a persisted **Haptics On/Off** setting.

---

## 5. SECURITY, ANTI-CHEAT & PERSISTENCE LAWS
- **Single Source of Truth State Machine:** All transitions (`Menu -> Playing -> Paused -> Win/Lose -> Retry -> Next`) and progression rules are strictly validated in `core/`.
- **Anti-Exploit Guards:**
  - Locked levels/content cannot be skipped or accessed via invalid state transitions.
  - All reward, currency, claim, purchase, and win-trigger actions must be **idempotent / debounced**—rapid multi-tapping must never duplicate rewards or trigger double wins.
- **Tamper-Resistant Atomic Saves:**
  - Save data must include a `schemaVersion` for safe migrations.
  - Protect local progress/currency with an integrity checksum/HMAC. If a save file is corrupted or tampered with, handle recovery gracefully without crash loops.
  - Write saves atomically so force-closing mid-write never wipes player progress.
- **Security & Offline Hygiene:**
  - Zero hardcoded secrets, API keys, or debug backdoors in client code.
  - Request zero unnecessary device permissions; default to zero hidden tracking.
  - **100% Offline-First:** Core gameplay must boot and run flawlessly with airplane mode enabled.

---

## 6. PERFORMANCE & APP LIFECYCLE LAWS
- **Frame Budget:** Target locked `60 FPS` (120 FPS where supported). Avoid per-frame heap allocations in hot render/update loops.
- **Resource Hygiene:** Use object pooling for particles/projectiles/repeated entities. Explicitly dispose listeners, timers, intervals, animation frames, audio nodes, and GPU textures on scene teardown—zero memory leaks across repeated restarts.
- **Lifecycle Resilience:**
  - On backgrounding, screen lock, or incoming call: auto-pause active gameplay, persist state, and mute audio.
  - **Delta-Time Clamping:** Clamp frame `deltaTime` (e.g., max `50ms`–`100ms`) so resuming from background never causes physics explosions or timer jumps.

---

## 7. MANDATORY VERIFICATION GATE (DEFINITION OF DONE)
Before reporting ANY task, feature, or bug fix as complete, you MUST:
1. **Write/Update Tests:**
   - Every new mechanic gets unit tests in `core/`.
   - Every bug fix gets an automated **regression test** proving the exact bug cannot recur.
2. **Run Full Local Gate:** Execute `TYPECHECK`, `LINT`, `TEST`, and `BUILD` commands defined in Section 1.
3. **Zero-Error Standard:** All commands must pass with **0 errors and 0 failing tests**. If any check fails, fix it and re-run the entire gate.
4. **Report Evidence Only:** Summarize exact files changed, tests added, and the real command outputs observed—never claim verification you did not execute.

---

## 8. REPOSITORY HYGIENE
- Keep the project root clean; place design/QA reports inside `docs/` and never dump temporary scratch files or logs into tracked directories.
- Update `docs/QA_AUDIT_REPORT.md` whenever resolving audit defects.