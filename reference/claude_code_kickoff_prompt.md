# Claude Code kickoff prompt — Beyond the Hollow (merge)

*(Paste everything below the divider into Claude Code, and put all of these files in the project folder first. The originals are essential — they're the actual game you're porting from.)*

**Attach / place in project:**
1. `claude_code_kickoff_prompt.md` — this prompt
2. `beyond_the_hollow_merge_spec.md` — architecture + story + phased plan + "definition of done"
3. `design_planet_a_kessas_reach.md` — Planet A feature bible (every system + tuning value)
4. `design_planet_b_hollow_verge.md` — Planet B feature bible (every system + tuning value)
5. `beyond_the_hollow.html` — the scaffold to build ON (do not restart)
6. `world_prototype.html` — **Planet A original (~4,150 lines) — the real Planet A code**
7. `planet_b_prototype.html` — **Planet B original (~2,000 lines) — the real Planet B code**

---

You're helping me finish a browser-based Three.js game called **Beyond the Hollow**. It currently exists as two separate, feature-complete prototypes that only communicate through browser-storage keys. The goal is to **merge them into one unified game that plays as a single, continuous story** — crash-land on Planet A ("Kessa's Reach"), survive and build a colony, get summoned to Planet B ("The Hollow Verge"), pass the three Threshold trials, reach the ending — with no "open a second file" seam in the middle.

## How to use the documents (read in this order)

1. **`beyond_the_hollow_merge_spec.md` first** — it's the authoritative plan: the sequential story, the unified `Engine.*` / `WorldManager` architecture, the six-phase port plan with acceptance criteria, and the definition of done. If anything conflicts, the spec wins on architecture/plan.
2. **The two per-planet bibles** (`design_planet_a_*.md`, `design_planet_b_*.md`) — exhaustive descriptions of *what to port*: every system, mechanic, enemy stat, weapon table, structure cost, trial spec, flag value, and tuning number, extracted from the real code. Each ends with a **port checklist** — tick every box as you go so nothing is lost.
3. **The two originals** (`world_prototype.html`, `planet_b_prototype.html`) — the ground truth for exact behavior, tuning, and every line of dialogue. The bibles tell you *what* exists and the numbers; the originals are *how* it's implemented. Port from these; use the bible as your checklist and the spec as your architecture.
4. **`beyond_the_hollow.html`** — the scaffold you build on. It already implements all the shared systems (below). Read it before porting.

## What the scaffold already does — reuse, don't rebuild

One renderer / camera / composer / clock / `animate()` loop, plus: `Engine.audio`, `Engine.rig`, `Engine.input` + `Engine.keybinds` (remappable controls), `Engine.player` (shared 3rd-person controller + camera + melee), `Engine.settings` (difficulty, colorblind, graphics, text size, subtitles), `Engine.storage`, `Engine.choicelog` (decisions + lore log), `Engine.ashketh` (voice overlay + subtitles), `WorldManager` (in-app world swap), and the start/character-customization screen, map, and armory. A visible red **error banner** surfaces startup failures — keep it.

## What is NOT built yet — this is the work (spec §4 + the bibles' checklists)

- **Planet A content → `WorldA`**: terrain/ruins/flora, colony build menu + resources, companions + branching dialogue (setting the decision flags), rival encounter, full weapon fire (ranged/explosive), ship flight + fuel, Readiness, day–night.
- **Planet B content → `WorldB`**: ring-world geometry, guardian constructs + mutated creatures, the three trials (Strength / Restraint / Cooperation), footholds, Ashketh musings, the four-tier ending.
- **The summon handoff**: replace Planet A's `window.open('planet_b_prototype.html')` with `Engine.goto('hollowVerge')`, carrying appearance, weapon, and the decision flags live in memory.
- **Dynamic dialogue**: keep the silent static fallback; for live generation outside the Claude.ai sandbox, wire a real API key **server-side** (never in the browser).

## Decision-flag values (use these exact strings — they're what the code checks)

- `rivalPath`: `truce` (recruits Kade) — otherwise hostile
- `reyesPath`: `confronted` | `patient`
- `oseiChoice`: `discovery_share` — otherwise suppressed (+ boolean `oseiDiscoveryDone`)
- `visionSeen`: boolean
- Ending trust score = +1 each for `rivalPath==='truce'`, `oseiChoice==='discovery_share'`, `reyesPath==='patient'` → four tone tiers.

## How I want you to work

- Go **phase by phase** per spec §4. Don't jump ahead.
- After each phase, **run it in a browser and confirm the acceptance criteria**, and tick the relevant boxes in that planet's bible checklist. Tell me what you tested.
- **Reuse `Engine.*`** — never re-duplicate audio, rig, input, combat, camera, or settings. Route input through named keybind actions. Push every major decision into `Engine.choicelog.record(...)` and lore into `Engine.choicelog.addLore(...)`.
- Keep changes **small and testable**; when ambiguous, ask rather than guess.

## Known gotchas (we already hit these — don't repeat them)

- Three.js is the **r128 global build** (CDN `<script>`, not modules). `THREE.CapsuleGeometry` does **not** exist in r128 — use `CylinderGeometry`.
- World modules expose `player` / `playerParts` via **getters** — don't assign to them.
- **`window.storage` only works in the Claude.ai sandbox.** For a standalone/hosted build it won't exist; the wrapper falls back gracefully. Don't make within-session continuity depend on it — state is shared live in memory now.

## Before Phase 1, confirm two things with me

1. **Target shape:** keep the single self-contained HTML file (recommended — preserves preview + the dynamic-dialogue sandbox; a later split into modules/bundler is mechanical), or go straight to a bundler now?
2. **Start point:** Planet A first (bigger, it's the entry world), or Planet B first (smaller, self-contained trials) to validate the world-module interface?

Start by reading the spec and the scaffold, confirm all seven files are present (especially the two originals), ask me those two questions, then begin. Finish line = the "Definition of done" in spec §8: one continuous playthrough, no reload, no second tab, decisions echoing into the ending, and every box in both bible checklists ticked.
