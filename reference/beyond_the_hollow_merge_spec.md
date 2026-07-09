# Beyond the Hollow — Unified Build & Story Specification

**Purpose of this document.** This is the single authoritative brief for merging the two existing prototypes of *Beyond the Hollow* — Planet A (`world_prototype.html`) and Planet B (`planet_b_prototype.html`) — into one unified codebase that plays as a single, continuous story. It is written to be handed directly to **Claude Code** as the build spec. It contains: the complete story told in sequential order; the unified engine architecture (already scaffolded — see "Provided starting artifact"); a phased port plan with acceptance criteria; and the specification for the remaining feature work (choice log + lore, deeper accessibility, dynamic dialogue).

**How to use this with Claude Code.** Start from the provided `beyond_the_hollow.html` scaffold (the "Engine shell"). Do not start the merge from the two original files directly — port their *content* into the shell's world modules, following Section 4. Work one phase at a time; each phase has an acceptance test that should pass in a browser before moving on.

---

## 1. What the game is (one paragraph)

*Beyond the Hollow* is a browser-based (Three.js) narrative sci-fi survival/exploration game across two connected worlds. The player commands a small expedition crew whose ship crashes on a hostile frontier world; they survive, build a colony, and — by proving themselves — earn passage to a second, vast alien megastructure ruled by an ancient AI custodian, **Ashketh**. Tone is epic and hopeful, never grimdark: a found-family crew building something against the odds. The two named companions are **Reyes** (pilot/mechanic) and **Dr. Amara Osei** (scientist/xenobiologist). A recruitable rival leader, **Kade**, can join depending on player choice.

---

## 2. The complete story, in sequential order

This is the intended single playthrough arc, start to finish. The merged build should present it as one continuous experience — no "open a second file" seam between the two halves.

### Act 0 — The Crash (cold open)
The expedition ship's course changes inexplicably in the final moments before descent — the player does not yet know this was **Ashketh's** unseen influence, drawing the crew in to observe how they handle adversity. The ship goes down on **Kessa's Reach**. The commander (the player) wakes; **Reyes** and **Osei** survived. The wrecked hull, sitting near the crash site, will become both the seed of the colony and — later — the ship that carries the crew onward. Character creation happens here (size, build, suit/leg/skin color, headgear); this appearance carries through the entire game.

### Act 1 — Kessa's Reach: survival and foothold
The core loop: leave the colony, explore forest, ruins, and open plains, fight hostile wildlife, gather **Wood** and **Scrap Metal**, return to build and grow the colony, and raise **Readiness** (a visible summary blending combat, exploration, and colony strength). Companion friction drives dialogue — Reyes pushes to fortify and keep everyone alive; Osei pushes toward the ruins and the mystery. Three of the game's four big decisions land during this act.

**Decision 1 — The Rival Faction: Fight or Fold Them In.**
Trigger: the crew encounters the rival human faction while *both sides* are under attack by wildlife — a shared crisis forcing an immediate call.
- *Fight past them* → a "purer" colony fully under the player's control, but a lasting rival threat.
- *Extend a truce* → a stronger colony; **Kade** is recruited and provides an ongoing mechanical contribution.
- Flag: `rivalPath` (`fight` | `truce`).

**Decision 2 — The Crash: Confront What Reyes Knows.**
Reyes has been hiding something about why the ship's course changed.
- *Push for an immediate confession* → answers early, but risks trust.
- *Let Reyes reveal it in his own time* → builds trust, delays key information.
- Flag: `reyesPath` (`pushed` | `waited`).

**Osei's vision moments (foreshadowing).**
Rare and wordless: once or twice, near a major ruin, Osei (given her sensitivity to the anomalies) experiences a brief presence — a pressure, a held breath, a half-heard tone. Never explained until Planet B. Keep these minimal and eerie so the Planet B reveal still lands.

**Decision 3 — Osei's Discovery: Share It or Suppress It.**
Osei finds hard evidence the crash wasn't an accident and something is watching the colony.
- *Tell the whole colony* → morale risk, but builds trust and unity.
- *Keep it contained to the crew* → avoids panic, isolates the player, deepens the crew's private burden.
- Flag: `oseiChoice` (`shared` | `suppressed`).

**Mid-game reward — The Ship.**
At **Readiness 50%**, Reyes announces he can get the wreck flying again — rebuilt from colony resources and the same hull that stranded the crew (a deliberate narrative full-circle). The ship is transport plus a forward-mounted weapon: full 6-directional flight (pitch/yaw/roll), a limited energy/fuel supply that trickles back in flight and refills fully when docked at the colony beacon.

**The Summon (the seam between worlds).**
At **Readiness 100%**, Ashketh actively *pulls the crew in* — a scripted summoning sequence near the ruins. The player does not choose to travel; they are chosen. **In the merged build this is the single most important change to the old flow:** instead of opening `planet_b_prototype.html` in a new tab, the summon triggers an in-app world swap (`Engine.goto('hollowVerge')`) that carries all decision flags and the player's appearance/loadout live in memory.

### Act 2 — The Hollow Verge: the Threshold
A colossal ancient **ring-world megastructure** built by the vanished **Architects**, who destroyed themselves through their own unchecked power. Colder, more mechanical tone than Kessa's Reach. Two threats mirror Planet A's dual-danger structure:
- **Guardian constructs** — corrupted Architect defense tech; disciplined, ranged, systemic (patrols/sentries).
- **Mutated creatures** — failed Architect experiments; fast, feral melee.

Base-building is reframed as **footholds** rather than a colony: a **Waypoint Beacon** (fast travel + stamina restore) and a **Repair Station** (passive HP regen), instant-construct, no separate resource economy. Throughout the act, Ashketh is present, and its dialogue **echoes the player's Planet A choices** (`rivalPath`, `reyesPath`, `oseiChoice`).

**Decision 4 — The Threshold: Answering Ashketh's Test.**
Played as a sequence of trial chambers, each testing a value Ashketh cares about. Ashketh isn't grading raw strength — it's grading whether the crew has the restraint, cooperation, and judgment the Architects lacked. Three trials (all retry-on-fail, no permanent lockout):
1. **Trial of Strength** — a 3-wave combat gauntlet.
2. **Trial of Restraint** — reach an altar without striking back at provocateurs; retaliating fails the trial.
3. **Trial of Cooperation** — guide an ally construct (the **Warden Fragment**) to hold one beacon while the player simultaneously holds another; attacking the ally fails the trial.

### Ending — "The Threshold"
One ending, but its **tone and detail shift across tiers** based on how the three tracked decisions leaned — toward **trust/cooperation** vs. **self-reliance/control**. Ashketh measures whether the crew is worthy of what the Architects could not handle. The throughline stays hopeful: the crew has proven something, and the door beyond the Hollow opens. Earlier choices should be *felt* here — echoed in Ashketh's framing and the ending text — paying off "choices shape the story" at the highest-stakes moment.

---

## 3. Provided starting artifact — `beyond_the_hollow.html` (the Engine shell)

A working, single-file scaffold is provided alongside this doc. **This is where the merge starts.** It already unifies everything that was duplicated across the two prototypes and demonstrates the target architecture with two thin placeholder worlds. Content porting happens into the labeled `PORT:` seams inside it.

What the shell already provides (all shared, declared once):

| System | Where | Replaces (old, duplicated) |
|---|---|---|
| Renderer / camera / composer / bloom / grade / clock / single `animate()` loop | top of script | two separate renderers + loops |
| `Engine.state` | shared core player state | overlapping `state` objects in both files |
| `Engine.storage` | async wrapper over `window.storage`, with fallback | raw `window.storage` handshake |
| `Engine.audio` | procedural SFX + per-world ambient mood | `ensureAudio`/`playTone`/… in both |
| `Engine.rig` | `makeHumanoidParts`, `CC_SIZES`, `applyCharacterConfig` | duplicated humanoid factories |
| `Engine.input` | keyboard + drag-look, **routed through remappable keybinds** | duplicated input handlers |
| `Engine.keybinds` | action→key map, persisted (`keybinds:v1`) | *(new)* |
| `Engine.player` | shared 3rd-person controller + camera + melee combat | `updatePlayer`/`updateCamera`/`performAttack` in both |
| `Engine.settings` | difficulty, colorblind, graphics, **text size, subtitles, controls** + panel | duplicated settings UIs |
| `Engine.choicelog` | decisions + lore fragments + panel (`choicelog:v1`) | *(new — see Section 5)* |
| `Engine.ashketh` | cinematic voice overlay, mirrors to subtitles | separate overlays |
| `WorldManager` | `register()` / `goto()` — in-app world swap | `window.open('planet_b_prototype.html')` |

**World-module interface** (each world implements this and plugs into `WorldManager`):

```
{
  id, name, mood,                 // mood = { tint, exposure, audio }
  scene,                          // the world's own THREE.Scene
  build(),                        // construct geometry, player, entities (once)
  enter(), exit(),                // lifecycle on activation/deactivation
  update(dt),                     // per-frame; calls Engine.player.update/updateCamera
  heightAt(x, z),                 // terrain height (flat for the ring-world)
  getTargets(),                   // combat targets for the shared melee system
  player,                         // the player mesh (getter)
  onInteract?(), onKill?(t), onDeath?()   // optional world hooks
}
```

**Note:** the shell's two worlds are intentionally near-empty (a few shapes + one dummy enemy each) to prove the architecture — shared character, movement, combat, settings, choice log, and live world-switching. The two dev buttons at the bottom are temporary; the real switch is the Ashketh summon.

---

## 4. Port plan (phased, each independently testable)

Port **content** out of the two original files into the shell's world modules. Reuse `Engine.*` for anything shared — do not re-duplicate audio, rig, input, combat, camera, or settings.

### Phase 1 — Kessa's Reach environment + player (WorldA)
Port from `world_prototype.html`: procedural grass/ground textures and rolling terrain (`heightAt`, `makeGrassTexture`), trees/alien flora/rocks/glow-flora, the ruins cluster (`addRuinPillar/Wall/Arch`, `RUINS_CENTER`), crash site, sky dome, day-night cycle (`updateDayNight`), ambient particles and wind sway, distant planet and stars.
- **Acceptance:** WorldA renders the full Kessa's Reach environment with day-night working; the player (shared rig, carried appearance) walks the real terrain; frame rate is stable.

### Phase 2 — Colony, resources, companions, rivals, weapons (WorldA)
Port: dual-resource system (Wood + Scrap), resource nodes + respawns, build menu + structures + colony strength; companions Reyes/Osei/Kade (`makeCompanion`, idle animation, name tags); the full branching **dialogue system** (`showNode`, dialogue trees) wired so Decisions 1–3 set `Engine.state.flags.{rivalPath, reyesPath, oseiChoice}` **and** call `Engine.choicelog.record(...)`; the rival encounter (`triggerRivalEncounter`, fight/truce, Kade contribution); weapon armory + progression (`WEAPON_DEFS`, unlocks, `equipWeapon`, gun/rocket fire) writing `weapon:state`; Osei's vision moments (`triggerVision`); Readiness (`computeReadiness`, `updateReadinessUI`).
- **Acceptance:** Player can gather, build, raise Readiness, meet the rival faction and resolve Decision 1, and trigger Decisions 2–3 through dialogue; each decision appears in the Expedition Log; combat uses the shared system with real weapons.

### Phase 3 — The ship + the summon handoff
Port ship build/flight/fuel/weapon (`buildShip`, `updateShipFlight`, `updateShipCamera`, `boardShip/exitShip`, fuel recharge at beacon) and the ship-unlock at Readiness 50%. Then replace the old summon tail: `triggerSummonSequence()` should, at its conclusion, call **`Engine.goto('hollowVerge')`** instead of `window.open(...)`. All decision flags and loadout are already in memory — no storage round-trip needed.
- **Acceptance:** At 50% Readiness the ship is flyable; at 100% the summon plays and hands off *in-app* to the Hollow Verge with appearance, weapon, and all three decision flags intact.

### Phase 4 — The Hollow Verge (WorldB)
Port from `planet_b_prototype.html`: ring-world geometry (floor, glow-lines, pillars, debris), colder lighting/grade/ambient; Guardian constructs + mutated creatures (reusing the shared combat/enemy pattern); the three trials (`startTrial`/waves, Trial of Restraint gauntlet + provokers, Trial of Cooperation + Warden Fragment); footholds (Waypoint Beacon, Repair Station); ambient particles (keep the efficient GPU point-sprite approach — not per-mesh); Ashketh musings.
- **Acceptance:** All three trials are completable and fail/retry correctly; footholds construct and function; enemies behave per type; tone reads distinctly colder than Kessa's Reach.

### Phase 5 — Choice echo, ending, and seam polish
Wire the **choice echo**: trial-start dialogue and the ending read `Engine.state.flags` (live, in-memory — no re-fetch of `save:v1`). Implement the tiered ending. Then play the full arc end-to-end and fix seams (state carrying correctly, no leftover references to the old two-file boundary, HUD/settings/log consistent across both worlds).
- **Acceptance:** A full playthrough from crash to ending runs without a reload or a second tab; the ending's tone visibly reflects the three decisions; nothing depends on the `window.storage` handshake for *within-session* continuity.

### Phase 6 — Retire the storage handshake (cleanup)
Once state is fully shared in memory, the old cross-file keys (`character:config`, `weapon:state`, `save:v1` decision flags) become **optional cross-session persistence** (save/load), not the link between worlds. Keep a single save/load path (`save:v1` for the whole game); `save:planetb:v1` is no longer needed as a separate world bridge.

---

## 5. Requested feature additions

### 5.1 Choice log + lore fragments — *scaffolded in the shell, needs content*
**Status:** the shared system exists (`Engine.choicelog`) with a **Log** button, a decisions/lore panel, persistence (`choicelog:v1`), and working demo entries. **Remaining work:** during Phases 2/4, replace the demo calls with the real ones —
- every major decision calls `Engine.choicelog.record(id, label, outcome)` at the moment it resolves, so the player has a visible record of how their choices shaped the world (the doc's "visible consequence tracking");
- discoverable lore in the ruins (Kessa's Reach) and Architect conduits (Hollow Verge) calls `Engine.choicelog.addLore(id, title, text)` when found.
Design intent: story **flags** still live in `Engine.state.flags` for gameplay/echo; the choice log is the *player-facing surface* of those flags plus optional lore. Keep lore fragments sparse and evocative, consistent with the game's understated foreshadowing.

### 5.2 Deeper accessibility — *scaffolded in the shell*
**Status:** implemented in `Engine.settings` / `Engine.keybinds` —
- **Remappable controls**: action→key map, click-to-rebind UI, reset-to-defaults, persisted (`keybinds:v1`); the shared controller reads through it (`Engine.input.down('forward')`, etc.). When porting Planet A/B input-driven actions, route them through named keybind actions rather than hard-coded codes.
- **Subtitles**: toggle that mirrors Ashketh's spoken lines to an on-screen caption. Extend this to any other narrated/dialogue text as it's ported.
- **Text size**: small/normal/large via a `--text-scale` CSS variable; apply it to any new text UI you add.
Difficulty scaling, colorblind mode, and graphics quality carry over from the originals and remain in the shared panel.

### 5.3 Dynamic dialogue — *not in the shell; port with care + verify*
The originals generate ambient companion/Ashketh lines live via a browser `fetch("https://api.anthropic.com/v1/messages")`, falling back instantly and silently to hand-written variant lines. **Important:** that fetch only succeeds inside Claude.ai's artifact-preview sandbox, which proxies the call without an API key. Once the game is opened standalone (downloaded, hosted, or run in a Claude Code dev environment), that proxy does not exist and the fetch fails — but this is already handled gracefully (timeout + try/catch → static fallback lines), so nothing breaks.
- **To keep live generation working after the merge**, wire a real API key **server-side** (a small backend proxy endpoint the client calls), never in the browser. Treat the static fallback lines as the source of truth for tone.
- **Open verification task (for the human, in artifact preview):** confirm a dynamic line actually fires end-to-end before relying on it. This has not been observed running live in this handoff — verify, don't assume.

---

## 6. Shared reference — state, flags, storage

**Decision flags** (live in `Engine.state.flags`, drive echo + ending):
- `rivalPath`: `fight` | `truce`
- `reyesPath`: `pushed` | `waited`
- `oseiChoice`: `shared` | `suppressed`

**Ending tiering:** score the three flags on a trust/cooperation ↔ self-reliance/control axis; select ending tone tier by how many lean which way (the originals used four tiers). Keep one broad outcome; vary framing and detail.

**Storage keys after the merge:**
- `save:v1` — the whole game's save/load (position, progress, flags, colony/trial state).
- `settings:v1` — accessibility settings (difficulty, colorblind, graphics, text size, subtitles).
- `keybinds:v1` — remappable controls.
- `choicelog:v1` — decisions + lore for the Expedition Log.
- `character:config`, `weapon:state` — optional; fold into `save:v1` or keep for cross-session convenience. The separate `save:planetb:v1` bridge is retired.

---

## 7. Architecture / target-shape decisions (confirm before Phase 1)

1. **Single-file vs. bundler.** The scaffold is a single self-contained HTML file using CDN-global Three.js r128 and `window.storage`. This preserves artifact-previewability and the dynamic-dialogue sandbox behavior. Recommendation: complete the *logical* unification (Phases 1–6) in this single-file form first; a later split into ES modules / a bundler is then a mechanical refactor. **Decide whether Claude Code should target single-file or go straight to a bundler.**
2. **Three.js version.** Both originals and the shell use r128 (global build, note: no `CapsuleGeometry`). If upgrading to a modern module build, budget for import/API changes across all ported systems.
3. **Save format.** Unify on one `save:v1` covering the whole game (Section 6).

---

## 8. Definition of done — "one seamless story"

The merge is complete when all of the following hold in a single browser session, with no reload and no second tab:

- Character creation → Kessa's Reach survival/colony loop → Decisions 1–3 (each recorded in the Expedition Log) → ship at Readiness 50% → summon at Readiness 100% → **in-app** handoff to the Hollow Verge → all three trials → tiered ending.
- Appearance, equipped weapon, and all three decision flags carry across the summon **in memory** (not via a storage handshake).
- Ashketh's Hollow Verge dialogue and the ending visibly reflect the three Planet A decisions.
- Choice log + lore, remappable controls, subtitles, and text size work across both worlds.
- Dynamic dialogue either works (server-side key wired) or falls back silently to static lines; the fallback path is confirmed.
- No duplicated engine systems remain: audio, rig, input, combat, camera, and settings each exist once, in `Engine.*`.

---

## 9. Source inventory

- `beyond_the_hollow.html` — **provided** unified Engine shell (start here).
- `world_prototype.html` (Planet A, ~4,150 lines) — content source for Phases 1–3.
- `planet_b_prototype.html` (Planet B, ~2,000 lines) — content source for Phase 4.
- `beyond_the_hollow_design_doc.md` — original story/design bible.
- `beyond_the_hollow_handoff_summary.md` — prior implementation-state summary.
- This document — the merge/build spec and single source of truth for the unified target.
