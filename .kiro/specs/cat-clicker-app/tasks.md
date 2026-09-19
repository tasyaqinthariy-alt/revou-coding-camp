# Implementation Plan: Cat Clicker App

## Overview

Build the entire Cat Clicker App as a single self-contained `index.html` file using HTML5, Tailwind CSS (CDN), and Vanilla JavaScript. The implementation follows the event-driven architecture defined in the design: Tailwind config block → static HTML shell → PetCounter → AuraCounter → Sound_Manager → Reaction_Engine → event wiring.

Each task is scoped to a discrete, testable piece of the file so that the app is runnable and incrementally verifiable after every step.

---

## Tasks

- [x] 1. Scaffold `index.html` — page shell, fonts, Tailwind CDN, and custom animation config
  - Create `index.html` with the full-viewport pastel gradient background (`#FFDEE9` → `#B5FFFC`)
  - Add Google Fonts `<link>` for Quicksand/Varela Round with a `sans-serif` fallback
  - Add the inline `<script>` Tailwind config block **before** the Tailwind CDN `<script>` tag, defining:
    - `wiggle` keyframe (0 % → 25 % → 50 % → 75 % → 100 % with correct `translateX` stops)
    - `zzzFloat` keyframe (opacity 0 → 1, translateY 0px → -30px)
    - `animate-wiggle` utility (0.6 s, ease-in-out, 1 iteration)
    - `animate-zzz` utility (1 s, ease-out, 1 iteration)
  - Add the Tailwind CDN `<script>` tag
  - Apply flexbox centering to the page body so the main container is horizontally and vertically centred
  - _Requirements: 1.1, 1.2, 1.3, 8.1, 8.2, 8.3, 8.4, 8.5_

- [x] 2. Render Cat_Element (inline SVG) and glassmorphism UI shell
  - [x] 2.1 Add the Cat_Element inline SVG
    - SVG must include a face outline, two ears, and two eyes
    - Set `tabindex="0"`, `aria-label="Pet the cat"`, `role="img"`, `style="cursor: pointer"`
    - Fix size at a square between 200 px – 400 px; maintain 1:1 aspect ratio
    - Idle state: no animation class, no CSS transform applied
    - _Requirements: 1.3, 1.4, 2.1, 2.2, 2.3, 11.1, 11.3_
  - [x] 2.2 Add Text_Overlay div and counter containers
    - Add `<div id="text-overlay">` with `aria-live="polite"` and `role="status"`; hidden by default
    - Add Pet_Counter display element with glassmorphism styling (semi-transparent white 30 % opacity, backdrop blur, pill corners, semi-transparent white border 40 % opacity)
    - Add Aura_Counter display element
    - _Requirements: 1.5, 10.2, 10.3, 11.4_

- [x] 3. Implement `PetCounter` module in a `<script>` block
  - [x] 3.1 Write the `PetCounter` object
    - `value` property initialised to 0
    - `init()`: reads `localStorage['cat-clicker-pet-count']`; validates with `Number.isInteger(raw) && raw >= 0`; defaults to 0 on any invalid/missing value; wraps localStorage access in `try/catch`
    - `increment()`: adds 1 to `value`, writes to `localStorage`, calls `updateDisplay()`
    - `updateDisplay()`: sets counter element `textContent` to result of `format(value)`
    - `format(n)`: pure function returning `"Total Pets: " + n + " ✨"`
    - Call `PetCounter.init()` on page load
    - _Requirements: 10.1, 10.2, 10.4, 10.5_
  - [ ]* 3.2 Write property test for `PetCounter.format` (Property 7)
    - **Property 7: Counter display text formatting**
    - **Validates: Requirements 10.2**
    - Use `fc.integer({ min: 0, max: 1_000_000 })` — assert `format(n) === "Total Pets: " + n + " ✨"`
    - Tag: `// Feature: cat-clicker-app, Property 7: Counter display text formatting`
  - [ ]* 3.3 Write property test for `PetCounter.increment` invariant (Property 3)
    - **Property 3: Pet_Counter increment invariant**
    - **Validates: Requirements 3.2**
    - Use `fc.integer({ min: 0, max: 1_000_000 })` — assert `value === n + 1` after one `increment()` call
    - Tag: `// Feature: cat-clicker-app, Property 3: Pet_Counter increment invariant`
  - [ ]* 3.4 Write property tests for `PetCounter.init` with localStorage round-trip and invalid values (Properties 5 & 6)
    - **Property 5: localStorage round-trip for Pet_Counter**
    - **Property 6: Invalid localStorage value defaults to 0**
    - **Validates: Requirements 10.1, 10.4, 10.5**
    - P5: `fc.integer({ min: 0, max: 1_000_000 })` — write via `increment()`, read via fresh `init()`, assert equality
    - P6: `fc.oneof(fc.string(), fc.constant(null), fc.float(), fc.integer({ max: -1 }))` — assert `init()` sets value to 0
    - Tag: `// Feature: cat-clicker-app, Property 5: localStorage round-trip for Pet_Counter` and `Property 6`

- [x] 4. Implement `AuraCounter` module
  - [x] 4.1 Write the `AuraCounter` object
    - `value` property initialised to 0
    - `increment(amount)`: adds `amount` to `value`, calls `updateDisplay()`
    - `updateDisplay()`: sets aura display element `textContent`
    - _Requirements: 4.5_
  - [ ]* 4.2 Write property test for `AuraCounter.increment` invariant (Property 4)
    - **Property 4: Aura_Counter increment invariant**
    - **Validates: Requirements 4.5**
    - Use `fc.integer({ min: 0, max: 1_000_000 })` — assert `value === n + 5` after `increment(5)`
    - Tag: `// Feature: cat-clicker-app, Property 4: Aura_Counter increment invariant`

- [x] 5. Implement `Sound_Manager` module
  - [x] 5.1 Write the `SoundManager` object
    - `play(assetKey)`: creates `new Audio(path)` for the given key (`'click'`, `'purr'`, `'zoomies'`), calls `.play()`, appends `.catch(() => {})` to silently swallow all errors
    - `stop(instance)`: calls `instance.pause()` and resets `instance.currentTime = 0`
    - Store asset key-to-path mappings as a plain object
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_
  - [ ]* 5.2 Write unit tests for `SoundManager` error handling
    - Verify reaction completes when `Audio.play()` rejects (mock `Audio` to return a rejected Promise)
    - Verify no user-visible error is shown
    - _Requirements: 7.6, 9.5_

- [x] 6. Implement `ReactionEngine` — `selectReaction` pure function
  - [x] 6.1 Write `selectReaction(n)` pure function
    - Maps integer 1–40 → `'happy'`, 41–70 → `'purring'`, 71–90 → `'sleeping'`, 91–100 → `'zoomies'`
    - Must be a standalone pure function with no side effects
    - _Requirements: 4.1, 5.1, 6.1, 7.1_
  - [ ]* 6.2 Write property tests for `selectReaction` (Properties 1 & 2)
    - **Property 1: Reaction selection covers the entire input domain**
    - **Property 2: Reaction selection is deterministic and partition-correct**
    - **Validates: Requirements 4.1, 5.1, 6.1, 7.1**
    - Use `fc.integer({ min: 1, max: 100 })` — assert correct type per range; no null/undefined returned
    - Tag: `// Feature: cat-clicker-app, Property 1: Reaction selection covers the entire input domain` and `Property 2`

- [ ] 7. Implement `ReactionEngine` — reaction apply methods and cancel
  - [-] 7.1 Define `ReactionConfig` records for all four reactions
    - Encode `type`, `duration`, `animClass`, `transform`, `overlayText`, `soundKey`, `auraBonus` per the design table
    - Initialise `AppState`: `{ isIdle: true, activeReaction: null, activeTimers: [] }`
    - _Requirements: 3.1, 4.2, 4.3, 5.2, 5.3, 6.2, 6.3, 7.2, 7.3_
  - [~] 7.2 Implement `ReactionEngine.cancel()`
    - Clear all `setTimeout` IDs in `AppState.activeTimers`
    - Remove animation classes from Cat_Element
    - Remove any inline CSS transform from Cat_Element
    - Hide Text_Overlay (remove opacity transition classes, set hidden)
    - Reset `AppState.isIdle = true`, `AppState.activeReaction = null`
    - _Requirements: 3.3_
  - [~] 7.3 Implement `applyHappy()`, `applyPurring()`, `applySleeping()`, `applyZoomies()`
    - Each method: adds the correct `animClass` (or transform for Sleeping), shows Text_Overlay with `overlayText`, calls `SoundManager.play(soundKey)`, calls `AuraCounter.increment(auraBonus)` if `auraBonus > 0`
    - Sleeping reaction: render 3 sequential "Zzz..." elements with `animate-zzz` class
    - Purring reaction: store the `Audio` instance returned by `SoundManager.play('purr')` and schedule `SoundManager.stop(instance)` at 2000 ms
    - Schedule a `setTimeout` for cleanup after `duration` ms: remove animation classes/transforms, fade out Text_Overlay over the correct fade duration, then hide it
    - Push all `setTimeout` IDs into `AppState.activeTimers`
    - _Requirements: 4.2, 4.3, 4.6, 5.2, 5.3, 5.4, 5.5, 6.2, 6.3, 6.5, 7.2, 7.3, 7.5_
  - [~] 7.4 Implement `ReactionEngine.trigger()`
    - Set `AppState.isIdle = false`
    - Call `cancel()` if a reaction is already active
    - Generate `Math.floor(Math.random() * 100) + 1` to get integer 1–100
    - Call `selectReaction(n)` to get the reaction type
    - Dispatch to the appropriate `apply*()` method
    - Increment `PetCounter`
    - _Requirements: 3.1, 3.2, 3.3, 3.4_

- [~] 8. Checkpoint — wire events, verify idle state and counter
  - Attach `click` event listener to Cat_Element → calls `ReactionEngine.trigger()`
  - Attach `keydown` event listener to Cat_Element → calls `trigger()` when `key === 'Enter'` or `key === ' '`
  - Verify Cat_Element shows a visible focus ring on keyboard focus (`focus-visible` outline or equivalent)
  - Open `index.html` in a browser and confirm: cat renders, counter shows 0 (or persisted value), clicking triggers a visible reaction and increments counter
  - _Requirements: 3.1, 3.2, 11.1, 11.2, 11.5_

- [ ] 9. Implement Text_Overlay fade-out transitions and Zzz animation sequencing
  - [~] 9.1 Add CSS transition for Text_Overlay opacity fade-out
    - Apply a CSS `transition: opacity Xms` class before hiding the overlay so the fade is visible
    - Happy/Zoomies: 300 ms fade; Purring: 200 ms fade; Sleeping: immediate removal
    - _Requirements: 4.6, 5.5, 6.5, 7.5_
  - [~] 9.2 Implement sequential Zzz element rendering for Sleeping reaction
    - Render 3 `<span>` elements with `animate-zzz` class, staggered using `setTimeout` offsets
    - Remove all Zzz elements simultaneously when the Sleeping reaction ends
    - _Requirements: 6.3, 6.5_
  - [ ]* 9.3 Write unit tests for reaction cleanup
    - Assert animation classes removed after correct timeout for each reaction type
    - Assert Text_Overlay hidden after reaction ends
    - Assert Cat_Element returns to idle state (no animClass, no transform) within 1 second of reaction end
    - _Requirements: 2.4, 2.5, 3.4_

- [ ] 10. Final accessibility and styling polish
  - [~] 10.1 Verify all accessibility attributes are in place
    - `tabindex="0"`, `aria-label="Pet the cat"`, `role="img"` on Cat_Element
    - `aria-live="polite"`, `role="status"` on Text_Overlay
    - Visible focus indicator on Cat_Element (outline/ring via `:focus-visible`)
    - _Requirements: 11.1, 11.3, 11.4, 11.5_
  - [~] 10.2 Verify glassmorphism container styling
    - Semi-transparent white background at 30 % opacity, backdrop blur, pill-shaped corners, semi-transparent white border at 40 % opacity
    - Counter display format: `"Total Pets: {n} ✨"`
    - _Requirements: 1.5, 10.2, 10.3_

- [~] 11. Final checkpoint — ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.
  - Confirm `index.html` is fully self-contained (no external JS files, no build step)
  - Confirm all four reactions are reachable and visually correct
  - Confirm Pet_Counter persists across page reload

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- All code lives inside `index.html` — there are no separate `.js` files
- Property tests (tasks 3.2–3.4, 4.2, 6.2) require a minimal Vitest + fast-check setup alongside `index.html`; export pure functions (e.g. `selectReaction`, `PetCounter.format`) via `<script type="module">` or a companion `.test.js` file
- Each task references specific requirements for full traceability
- Checkpoints (tasks 8 and 11) ensure incremental validation at logical breaks

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["2.1", "2.2"] },
    { "id": 1, "tasks": ["3.1", "4.1", "5.1", "6.1"] },
    { "id": 2, "tasks": ["3.2", "3.3", "3.4", "4.2", "5.2", "6.2", "7.1"] },
    { "id": 3, "tasks": ["7.2", "7.3"] },
    { "id": 4, "tasks": ["7.4"] },
    { "id": 5, "tasks": ["9.1", "9.2", "10.1", "10.2"] },
    { "id": 6, "tasks": ["9.3"] }
  ]
}
```
