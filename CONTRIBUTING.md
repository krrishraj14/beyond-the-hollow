# Contributing to Beyond the Hollow

Thanks for your interest! This project is deliberately simple to hack on: the whole game is one HTML file with no build step.

## Getting started

1. **Fork** this repository (button at the top right on GitHub).
2. **Clone** your fork and create a branch:
   ```
   git clone https://github.com/<your-username>/beyond-the-hollow.git
   cd beyond-the-hollow
   git checkout -b my-feature
   ```
3. Open `beyond_the_hollow.html` in a browser to run the game. Edit, refresh, repeat — that's the whole dev loop.
4. Commit your changes and push the branch to your fork.
5. Open a **Pull Request** against `main` describing what you changed and why.

## Finding your way around the code

Everything lives in `beyond_the_hollow.html`, organized top to bottom:

- **`Engine.*`** — shared systems, each declared once: `storage`, `audio`, `rig` (character models), `keybinds`/`input`, `player` (movement/camera/jump), `combat` (melee/ranged/explosive), `settings`, `choicelog`, `ashketh` (voice overlay), `save`, `hints`, `armory`, `charcreate`, `map`.
- **`WorldA`** ("Kessa's Reach") and **`WorldB`** ("The Hollow Verge") — self-contained world modules that plug into `WorldManager` via a small interface (`build`, `enter`/`exit`, `update`, `heightAt`, `getCombatTargets`, ...).
- The design documents in `reference/` describe every system and tuning value — worth a skim before touching gameplay.

## Ground rules

- **Reuse `Engine.*`** — don't duplicate audio, input, combat, or camera logic inside a world.
- Match the existing code style; keep the game a single self-contained file (CDN scripts only).
- Three.js is the **r128 global build** — e.g. `THREE.CapsuleGeometry` does not exist; check the r128 docs.
- Test in a real browser before opening a PR: play the affected flow, and watch the console (a red banner appears on startup errors).
- Keep the tone of story text epic and hopeful, never grimdark — see the design docs.

## Reporting bugs / suggesting features

Open a GitHub **Issue** with steps to reproduce (for bugs) or a short pitch (for features). Screenshots welcome.
