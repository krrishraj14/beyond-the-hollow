# Beyond the Hollow — Planet B Design Bible: "The Hollow Verge"

*Complete feature/system specification for Planet B, extracted from the working prototype `planet_b_prototype.html` (~2,000 lines). Ground-truth description of every system, mechanic, and tuning value; exact text and full implementations live in the prototype itself. Pair with `beyond_the_hollow_merge_spec.md` and port into the scaffold's `WorldB` module.*

---

## 0. Role in the arc
The second world and the climax. An ancient **ring-world megastructure** built by the vanished **Architects**, now decaying and corrupted, watched over by **Ashketh**. The player, summoned here after proving themselves on Kessa's Reach, faces the **Threshold** — a sequence of trials testing the values the Architects lacked — then reaches the ending. Colder, mechanical palette. Danger: guardian constructs + mutated creatures.

## 1. Environment & atmosphere
- **Ring-world floor** (flat; `heightAt` returns 0) with **glowing floor lines** (`addFloorGlowLine`), structural **pillars** (`addPillar`, tall emissive columns), and scattered **debris** (`addDebris`).
- Cold lighting: dim hemisphere + a bluish key light; fog tuned cold/blue.
- **VFX pipeline** at parity with Planet A: same composer/bloom/ACES/grade, tuned **colder/bluer**. Ambient particles use an **efficient GPU point-sprite system** (`AMBIENT_COUNT = 90`, `updateAmbientParticles`) — *not* per-mesh particles (this was a real performance fix; keep it that way).
- **Audio:** deliberately more dissonant/mechanical than Planet A — ambient hum (`startAmbientHum`) + a colder procedural score (`startAmbientMusic`/`updateAmbientMusic`, combat-tension reactive). Full SFX set incl. a `beacon` chime.

## 2. Carried-over character & weapon
On load, `applyCarriedCharacterAndWeapon()` reads the shared `character:config` and `weapon:state` so the player's appearance and equipped weapon match Kessa's Reach. In the merged build this comes from live in-memory state instead of a storage round-trip.

## 3. Combat & threats (mirrors Planet A's dual-danger shape)
- **Player melee** identical constants to Planet A: range `2.4`, arc `cos(55°)`, damage `25`, cooldown `0.55`.
- **Guardian constructs** (`makeGuardian`): **stationary, ranged, disciplined.** `hp 45`. Fire projectiles at the player (`fireProjectile`) — travelling shots the player must avoid. Represent Architect defense tech.
- **Mutated creatures** (`makeMutant`): **fast melee chasers.** `hp 35`. Feral; close distance and strike. Failed Architect experiments.
- **Ambient roamers** spawn outside the trials so the structure feels inhabited; trials spawn their own controlled sets.
- Shared enemy update loop (`updateEnemies`), hurt flash, and retryable game over (`triggerGameOver`).

## 4. The Threshold — three trials (Decision #4, played out)
All trials **retry on fail** — no game over, no permanent lockout. Completion flags: `trialComplete`, `trial2Complete`, `trial3Complete`.

### Trial 1 — Trial of Strength (combat gauntlet)
- Center `TRIAL_CENTER = { x: 0, z: -28 }`. Three escalating waves, enemy counts `TRIAL_WAVES = [2, 3, 4]` (`startTrial` → `spawnTrialWave` → `checkTrialWaveClear`). Clear all waves to pass.

### Trial 2 — Trial of Restraint (a gauntlet, not a fight)
- Layout `TRIAL2 = { entranceZ: 37, altarZ: 49, x: 0, wallXOffset: 5 }`. The player must **reach the altar without striking back** at provokers (`makeProvoker`, `updateProvokers`) that harass along the corridor. **Retaliating fails the trial** (`failTrial2`). Tests self-control.

### Trial 3 — Trial of Cooperation (a trust puzzle, not a fight)
- Layout `TRIAL3 = { x: 34, z: 0, beaconA (34,-8), beaconB (34,8), entrance (22,0) }`. The player and an **ally construct — the "Warden Fragment"** (`makeWardenFragment`, `updateWardenFragment`) must **simultaneously hold two separate beacons** (`makeBeacon`) for `TRIAL3_HOLD_NEEDED = 2.5s`. The player signals/guides the ally (`signalWardenFragment`); **attacking the ally fails the trial** (`failTrial3`). Tests cooperation/trust.

## 5. Footholds — Planet B's base-building, re-skinned
Instant-construct, no resource economy. `FOOTHOLD_DEFS`: **Waypoint Beacon** (`0x6fd0e0`) and **Repair Station** (`0x6fe0a0`). Placed footholds: `fh_entrance` (waypoint) at (8,-10), `fh_mid` (repair) at (16,22). `constructFoothold` lights the pad/ring/beacon and confirms via Ashketh. Intended function: waypoint = fast travel + stamina restore; repair station = passive HP regen. Framing is "securing hostile ground," not "building a home."

## 6. Ashketh — presence, musings, choice echo
- **Dialogue overlay** (`showAshketh`) is the voice of the structure throughout.
- **Static musings** `ASHKETH_STATIC_MUSINGS`: occasional idle lines while exploring outside trials. Also supports **dynamic live-generated musings** (same API-fetch-with-silent-fallback design as Planet A; sandbox-only for live generation).
- **Choice echo** (`loadPlanetAFlags`): reads Kessa's Reach's save (`save:v1`) once and reuses it for trial-start dialogue **and** the ending. Pulls `rivalPath`, `reyesPath`, and Osei's choice (`npcState.osei`). In the merged build these come from live shared state.

## 7. The ending — "The Threshold"
- Gated by all three trials complete (`checkThresholdGate`, guarded by `thresholdTriggered`).
- **Trust score** (0–3), +1 for each of: `rivalPath === 'truce'`, `oseiChoice === 'discovery_share'`, `reyesPath === 'patient'`.
- **Four tone tiers** of ending text (title always "BEYOND THE HOLLOW"), selected by trust score:
  - **3** — you never stood apart from your people; Ashketh offers only respect.
  - **2** — you mostly chose trust; it held even here.
  - **1** — some doors opened together, some kept closed; both are a kind of strength.
  - **0** — you carried it alone; unbroken, on terms no one else set — its own kind of answer.
- Sequence: a staged cutscene (`overlay` dims, four Ashketh lines ~3s apart: "Three trials. Three answers…"), then the tiered title + ending text fade in. Retryable restart button. One broad outcome (the crew passes), tone shaped by choices — keeping the heroic, hopeful throughline.

## 8. Accessibility & settings (shared `settings:v1`)
Same three original toggles as Planet A — difficulty (`DIFFICULTY_MULT` easy `0.6` / normal `1.0`), colorblind mode, graphics quality — **sharing the same storage key** so choices carry across both worlds. *(Merge adds text size, subtitles, remappable controls — already in the scaffold.)*

## 9. Save system (`save:planetb:v1`)
`getSaveDataB()` persists: version, player x/z, hp, stamina, `trialComplete`, `trial2Complete`, `trial3Complete`, `thresholdComplete`, and `footholds[]` built state. Autosave debounced ~1.5s. **After the merge**, fold this into the single unified `save:v1` (merge spec Phase 6) — the separate Planet B save/bridge is retired.

## 10. Storage keys (current, pre-merge)
Reads `character:config`, `weapon:state`, `settings:v1`, and `save:v1` (for choice echo). Writes `settings:v1` (shared) and `save:planetb:v1` (its own progress).

---

## 11. Port checklist — Planet B → `WorldB` (tick each during the merge)
- [x] Ring-world geometry: floor + glow-lines + pillars + debris
- [x] Cold lighting + fog + colder-tuned grade/bloom
- [x] Efficient GPU point-sprite ambient particles (keep — do not use per-mesh)
- [x] Audio: dissonant ambient hum (per-world mood retune) + full SFX incl. beacon chime *(tension-reactive score simplified to the shared ambient bed — see scaffold notes)*
- [x] Carried character + weapon applied on entry (from live shared state)
- [x] Guardian constructs (stationary ranged, hp 45, `fireProjectile`)
- [x] Mutated creatures (fast melee, hp 35)
- [x] Ambient roamers outside trials
- [x] Trial 1 — Strength: waves `[2,3,4]`, retry on fail
- [x] Trial 2 — Restraint: reach altar without retaliating; provokers; fail-on-strike
- [x] Trial 3 — Cooperation: Warden Fragment + dual beacons, hold 2.5s, fail if ally attacked
- [x] Footholds: waypoint + repair, instant-construct, their functions (fast travel/stamina, HP regen)
- [x] Ashketh overlay + static musings + dynamic musings (silent fallback)
- [x] Choice echo from live flags into trial-start dialogue **and** ending
- [x] Ending: trust score (0–3) → four tiers, staged cutscene, retryable restart
- [x] Settings via shared `settings:v1`
- [ ] Save/load folded into unified `save:v1` (retire `save:planetb:v1`)
