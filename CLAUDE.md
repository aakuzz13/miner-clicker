# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`miner-clicker` is a single-file, self-contained incremental/clicker game ("Кристальная Шахта 2.0" — Crystal Mine 2.0), written in Russian for the UI text. The entire app — markup, CSS, and JavaScript — lives in `miner-clicker-v2.html`. There is no build system, no package manager, no dependencies, and no test suite. To work on the game, open/edit `miner-clicker-v2.html` directly and load it in a browser (double-click the file, or serve it with any static file server, e.g. `python3 -m http.server`).

## Architecture

Everything is in one IIFE inside the `<script>` tag at the bottom of `miner-clicker-v2.html`. Key pieces, in the order they appear:

- **State**: a single `state` object (resources, totalEarned, clickLevel, autoMiners array, currentLocation, muted) persisted to `localStorage` under `SAVE_KEY` via `load()`/`save()`. `save()` is called after most state-mutating actions and on an interval/`beforeunload`.
- **Static game data**: `clickUpgrade` (the pickaxe upgrade), `autoDefs` (the 4 auto-miner tiers with rarity/cost/rate), `LOCATIONS` (5 unlockable locations with theming colors and `unlockAt` thresholds based on `totalEarned`), and `RARITY` (label/icon per rarity tier).
- **Derived getters**: `costFor()`, `clickPower()`, `perSecond()`, `unlockedCount()`, `locationBonusMult()` — pure functions computed from `state` + static data, not cached, called wherever needed (nothing goes stale).
- **Audio**: a small synth built on `AudioContext` (`tone()`, `sweep()`, `playClick()`, `playBuy()`, `playTravel()`, `playUnlock()`) — no audio files, all procedurally generated oscillators/gains. Respects `state.muted`.
- **Rendering**: DOM is built imperatively (`buildMinerCard()`, `buildLocationCard()`, `renderClickTab()`/`renderAutoTab()`/`renderLocTab()`, dispatched by `renderPanel()` based on `activeTab`). There is no virtual DOM/diffing — `panel.innerHTML = ""` then full rebuild on every render.
- **Game loop**: a `setInterval` tick every 200ms applies `perSecond() * dt` earnings; a separate interval autosaves every 5s. `applyOfflineProgress()` grants earnings for time elapsed since the last visit (capped at 4 hours), read/written via a `SAVE_KEY + "_ts"` timestamp in `localStorage`.
- **Locations/theming**: switching location (`travelTo()`) drives a CSS custom-property-based re-theme (`--accent`, `--rockBase`, background gradient, etc.) plus a portal/flip transition animation; `locationBonusMult()` gives +15% global income per unlocked location beyond the first.
- **Effects**: floating numbers, particle "chips", and ambient background particles are spawned as short-lived DOM elements removed via `setTimeout`, not CSS animation event listeners.

## Conventions to preserve when editing

- Keep everything in the one HTML file — this project is intentionally dependency-free and buildless.
- All user-facing strings are in Russian; match the existing tone/style if adding new UI text.
- Respect `prefers-reduced-motion` — most animation/particle code checks the `reducedMotion` flag or has a corresponding `@media (prefers-reduced-motion: reduce)` CSS override; new animated effects should do the same.
- Game balance (costs, `costMult`, rates, `unlockAt` thresholds) lives entirely in the `clickUpgrade`/`autoDefs`/`LOCATIONS` data structures near the top of the script — tune numbers there rather than scattering magic numbers through logic.
