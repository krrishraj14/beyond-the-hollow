# Beyond the Hollow — Planet A Design Bible: "Kessa's Reach"

*Complete feature/system specification for Planet A, extracted from the working prototype `world_prototype.html` (~4,150 lines). This is the ground-truth description of every system, mechanic, and tuning value. Exact dialogue text and full implementations live in `world_prototype.html` itself — this document tells you what exists, how it behaves, and the numbers behind it, so nothing is lost or reinvented during the merge. Pair it with `beyond_the_hollow_merge_spec.md` (architecture + phased plan) and port into the scaffold's `WorldA` module.*

---

## 0. Role in the arc
The first world and the game's proving ground. The crew crash-lands here; the player survives, builds a colony, resolves three of the four big decisions, and raises **Readiness** until Ashketh summons them to Planet B. Temperate, overgrown ruins-and-wilderness. Warm palette. Danger: hostile wildlife + a rival human faction.

## 1. Environment & world
- **Terrain:** procedural rolling ground via a `heightAt(x, z)` function; 200×200 plane, displaced and re-normaled. Ground uses a **procedurally generated grass texture** (canvas) with a matching roughness map, repeat-wrapped.
- **Sky & atmosphere:** shader sky dome (top color `0x6a7268`, bottom `0xa89878`), fog (`0x8a8068`, near 14 / far 68), a field of ~1,400 stars, and a distant textured planet (radius 55) with emissive glow.
- **Day–night cycle:** `updateDayNight` drives sun height/color, hemisphere + directional (`0xcfc3a8`, intensity ~1.05) + backlight, sky tint, and ambient-music tension over `state.timeOfDay` (0..1). Player starts at `timeOfDay 0.3`.
- **Ruins cluster:** centered at `RUINS_CENTER = { x: 32, z: -9 }` — pillars, walls, arches (`addRuinPillar/Wall/Arch`). Site of Osei's vision moment.
- **Set pieces:** crash site (`buildCrashSite`) near the colony; broken comms dish (`buildCommsDish`) at ~(17,-4); ridge rocks; alien trees, normal trees, rocks, and glowing flora scattered across the map.
- **VFX pipeline:** post-processing composer with bloom (`UnrealBloomPass`), ACES tone mapping, sRGB, and a custom **cinematic grade** shader pass (advances over time). Rim-light injection on select materials (`addRimLight`, color `0xbfe0ff`). Ambient drifting particles (`createParticleSystem`/`updateParticles`), wind sway on foliage (`updateWindSway`), dust bursts on impacts (`spawnDustBurst`).

## 2. Character system (writes `character:config`)
- **Sizes** `CC_SIZES`: small `0.88`, standard `1.0`, large `1.14`.
- **Builds** `CC_BUILDS` (torso top/bottom radii): slim `0.27/0.31`, standard `0.32/0.36`, heavy `0.40/0.44`.
- **Suit colors** (8): `0x4f8ff0, 0xe2574c, 0x4fd18a, 0xe2b04c, 0x9b6fe0, 0x2a2f3a, 0xe08fc0, 0x6fe0e0`.
- **Leg colors** (6): `0x262b35, 0x4a3a2f, 0x3a3f4a, 0x5a2a2a, 0x2a4a3a, 0x1a1a1a`.
- **Skin tones** (5): `0xf0c89a, 0xd9a876, 0xb87f56, 0x8a5a3a, 0x5c3a26`.
- **Headgear** (toggle meshes on the head): `none`, `cap` (hemisphere, suit-colored), `helmet` (shell + emissive visor `0x6fd0e0`), `crest` (cone). *(Note: the scaffold's customization currently covers size/build/colors; headgear is the one piece to re-add during the port.)*
- `applyCharacterConfig(cfg)` sets scale, rebuilds torso geometry per build, and recolors torso/arms/legs/head + headgear. Config persists to `character:config` and carries to Planet B.

## 3. Movement, camera, traversal
- Third-person chase camera, `CAM_DIST ~6.5`, orbit yaw + pitch (`PITCH_HOVER_MIN/MAX ≈ ±1.48`), mouse-drag + touch look, virtual joystick on touch.
- Walk/sprint with **stamina** (sprint drains, idle regenerates); footstep SFX cadence scales with speed.
- **Jump**: `JUMP_FORCE 6.2`, `GRAVITY 17`, airborne offset + land event (dust + SFX).
- Player rig animates (arm/leg swing) while moving; talking/gesture animation during dialogue; death animation on game over.

## 4. Combat
- **Player melee base:** `PLAYER_ATTACK_RANGE 2.4`, arc ≈110° cone (`cos(55°)`), `PLAYER_ATTACK_DAMAGE 25`, `PLAYER_ATTACK_COOLDOWN 0.55`. Damage scaled by difficulty.
- **Weapons** `WEAPON_DEFS` (equipped model + stats; unlocks gate progression):

  | id | name | type | dmg | range | cooldown | unlock |
  |---|---|---|---|---|---|---|
  | stick | Branch | melee | 14 | 2.1 | 0.50 | always |
  | bat | Scrap Bat | melee | 24 | 2.3 | 0.68 | 3 kills |
  | sword | Fabricated Blade | melee | 32 | 2.6 | 0.42 | 8 kills |
  | gun | Sidearm | ranged | 20 | 22 | 0.32 | 40% Readiness |
  | rpg | Launcher | explosive | 65 | 30 | 1.70 | 75% Readiness (splash 3.6m) |

  Ranged uses a tight cone (`cos(8°)`), instant-hit tracers (`spawnTracer`/`updateTracers`), recoil (`updateWeaponRecoil`), muzzle position from the rig. Explosive fires a travelling rocket (`fireRocket`/`updateRockets`/`explodeRocket`) with splash. Armory UI lists cards with unlock progress; unlock toasts fire when thresholds are met. Weapon state persists to `weapon:state` (`{equipped, unlocked[]}`).
- **Wildlife enemies** (`makeEnemy`): `hp 40`, `ENEMY_AGGRO_RADIUS 9`, `ENEMY_ATTACK_RANGE 1.6`, `ENEMY_LEASH_RADIUS 11`; idle/chase/leash-return AI (`updateEnemies`). Initial spawn set: (10,10), (-14,8), (6,16), (-9,-14), (18,5), plus two already attacking the rival outpost at (-19,16), (-23,13). Killing enemies increments `state.enemiesDefeated`.
- Hurt flash + camera shake on player damage; `triggerGameOver` on death (retryable).

## 5. Colony & base-building
- **Resources:** `Wood` + `Scrap Metal`. Resource nodes (`makeResourceNode`) are gatherable (wood = sphere, scrap = octahedron) and **respawn** on a timer (`updateResourceRespawns`). Starting nodes: wood at (4,-3),(9,6),(-10,3),(-3,-9); scrap at (-15,-6),(13,-10),(-5,12).
- **Structures** `structureDefs` (build menu → `buildStructure` → `createStructureMesh` into build slots):

  | id | name | cost (wood/scrap) | strength | max count |
  |---|---|---|---|---|
  | shelter | Shelter | 5 / 0 | 2 | 3 |
  | storage | Storage Depot | 8 / 2 | 3 | 3 |
  | workshop | Workshop | 14 / 6 | 5 | 2 |

  Each build adds to `state.colonyStrength` (feeds Readiness). Build menu UI shows cost/availability; `builtCounts` + `buildSlots` track placement and persist in the save.

## 6. Companions & dialogue
- **Companions:** **Reyes** (pilot/mechanic), **Dr. Osei** (scientist), **Kade** (rival leader, only if recruited). Each has a floating name tag; idle animation; talking animation during dialogue.
- **Dialogue system** (`dialogueTrees`, `showNode`, `resolveStartNode`, `tryInteract`): each NPC has `displayName`, a `start` node, and `nodes{}`. Nodes carry `text`, optional `choices[]` (label → next), `next`, and `setFlag` (e.g. `reyesPath:confronted`). Start node resolves based on current flags so conversations advance across sessions (`npcState.{reyes,osei,kade}.current`).
- **Dynamic ambient dialogue:** repeatable "loop" nodes are `dynamicEligible` with a `personality` prompt and a `variants()` function returning context-aware fallback lines (react to `colonyStrength`, `shipUnlocked`, `enemiesDefeated`, etc.). Live generation calls `fetch("https://api.anthropic.com/v1/messages")`; on timeout/failure it falls back **silently** to the hand-written variant lines. **Live generation only works inside Claude.ai's artifact sandbox** — outside it, the fallback path runs (see merge spec §5.3 for the server-side key path).

## 7. Rival human faction
- Encounter zone `RIVAL_ENCOUNTER = { x: -20, z: 14, radius: 10 }`, triggered while both crew and rivals are under wildlife attack (`triggerRivalEncounter`).
- Rivals (`makeRivalHumanoid`, `hp 50`, attack range `1.6`): **Kade** (leader) at (-20,15) and a **scout** at (-22,13). AI in `updateRivalNpcs`.
- **Decision:** `resolveRivalFight()` vs `resolveRivalTruce()`.
  - **Truce** → Kade is recruited; ongoing colony contribution via `updateKadeContribution` (timer starts ~25s) which periodically adds to the colony.
  - **Fight** → rivals remain a hostile threat; purer, player-controlled colony.
- Sets flag `rivalPath` and records `rivalKadeDead/rivalScoutDead/rivalActive` in the save.

## 8. The four decisions (as they surface on Planet A)
1. **Rival faction — Fight or Fold In** → `rivalPath` (`truce` recruits Kade; otherwise hostile).
2. **The Crash — Confront Reyes** → `reyesPath` (`confronted` = pushed for the truth now; `patient` = gave him space).
3. **Osei's Discovery — Share or Suppress** → `oseiChoice` (`discovery_share` = told the colony; otherwise suppressed) and `oseiDiscoveryDone`.
4. **The Threshold** — plays out on Planet B (see the Planet B bible), but is colored by 1–3.
- **Vision moment** (`triggerVision`): a rare, wordless Ashketh presence near the ruins; sets `visionSeen`.

## 9. Ashketh's presence & the summon (the seam to Planet B)
- Fleeting vision on Planet A (above). No explanation until Planet B.
- **Summon:** when `computeReadiness().overall >= 100`, `checkAshkethSummon` → `triggerSummonSequence` plays a scripted cutscene (camera move to ruins, staged Ashketh lines) and — in the original — a button opens `planet_b_prototype.html`. **In the merged build this becomes `Engine.goto('hollowVerge')`**, carrying appearance, weapon, and all decision flags live (merge spec Phase 3).

## 10. The ship (mid-game reward)
- **Unlock:** `state.shipUnlocked` at **Readiness ≥ 50%** (`checkShipUnlock`); Reyes announces the rebuild. Parked at `SHIP_PARK = { x: 4, z: 18 }` (the crash hull).
- **Flight model** (`updateShipFlight`, 6-DoF): `SHIP_MAX_SPEED 22`, `SHIP_ACCEL 11`, `SHIP_DRAG 0.55`, pitch rate `1.1`, yaw rate `1.0`, roll rate `1.7`; chase camera (`updateShipCamera`).
- **Fuel:** drains under thrust (`SHIP_FUEL_DRAIN_THRUST 6`), trickles back in flight (`+1.5`), refills fast when docked at the beacon (`+26`). Max fuel 100.
- **Weapon:** forward-mounted gun (`fireShipWeapon`), strafes ground enemies/rivals from the air. Board/exit via interaction (`boardShip`/`exitShip`); must land to exit.

## 11. Readiness (the gate to Planet B)
`computeReadiness()` averages three tracks (each capped at 100):
- **Combat** = `enemiesDefeated / 8 × 100` (`READINESS_TARGETS.combatKills = 8`).
- **Exploration** = fraction of milestones met among `[visionSeen, rivalPath set, oseiDiscoveryDone, reyesPath set]` × 100.
- **Colony** = `colonyStrength / 24 × 100` (`READINESS_TARGETS.colonyStrength = 24`).
- **Overall** = average of the three. Shown as a clickable Readiness badge + a breakdown panel. Summon fires at 100.

## 12. Audio
- **SFX:** swing, hit, enemy death, player hurt, gather, build, blip, jump, land, gunshot, rocket launch, explosion (all procedural via `playTone`/`playNoiseBurst`).
- **Ambient:** wind bed (`startAmbientWind`) + a tension-reactive procedural score (`startAmbientMusic`/`updateAmbientMusic`, driven by sun height + combat tension). Footsteps cadence-matched to movement.

## 13. Accessibility & settings (shared `settings:v1`)
- **Difficulty** multiplier `DIFFICULTY_MULT`: easy `0.6`, normal `1.0` (scales damage taken/dealt).
- **Colorblind mode** (`applyColorblindMode`) and **graphics quality** toggle (`applyGraphicsQuality`, affects bloom/pixel ratio). Settings panel shared with Planet B via the same storage key. *(Merge adds text size, subtitles, remappable controls — already in the scaffold.)*

## 14. Save system (`save:v1`)
`getSaveData()` persists: version, player x/z, hp, stamina, wood, scrap, colonyStrength, enemiesDefeated, timeOfDay, shipUnlocked, summoned, a copy of all `flags`, `npcState.{reyes,osei,kade}.current`, `builtCounts`, build `slots[]`, and rival death/active state. Autosave debounced ~1.5s (`queueAutosave`). Planet B reads this file's `flags` + `npcState.osei` for choice-echo and the ending.

## 15. Decision-flag reference (use these exact values)
- `rivalPath`: `truce` (recruit Kade) | *(hostile if not truce)*
- `reyesPath`: `confronted` | `patient`
- `oseiChoice`: `discovery_share` | *(suppressed if not shared)*; plus boolean `oseiDiscoveryDone`
- `visionSeen`: boolean (ruins vision witnessed)

## 16. Storage keys written by Planet A
`character:config` (appearance), `weapon:state` (`{equipped, unlocked[]}`), `settings:v1` (shared), `save:v1` (full save + flags, read by Planet B).

---

## 17. Port checklist — Planet A → `WorldA` (tick each during the merge)
- [x] Procedural terrain (`heightAt`) + grass/roughness textures
- [x] Sky dome, fog, stars, distant planet, day–night cycle
- [x] Ruins cluster, crash site, comms dish, flora/rocks/glow-flora
- [x] Post-processing: bloom + ACES + cinematic grade + rim light + particles + wind sway + dust bursts
- [ ] Character system incl. **headgear** (writes `character:config`)
- [x] Movement, stamina/sprint, jump/gravity, footsteps, rig animation *(talking + death animation land with dialogue/game-over in Phase 2)*
- [x] Melee combat + full `WEAPON_DEFS` (melee/ranged/explosive) + tracers/recoil/rockets + armory UI + unlock toasts (`weapon:state`)
- [x] Wildlife enemies (params + spawn set + AI) + hurt flash/shake + game over
- [x] Resources (wood/scrap), resource nodes + respawn, `structureDefs` + build menu + colony strength
- [x] Companions (Reyes/Osei/Kade) + name tags + full `dialogueTrees` + branch flags
- [x] Dynamic ambient dialogue (dynamicEligible/personality/variants + silent fallback)
- [x] Rival encounter (fight/truce, Kade recruit + contribution)
- [x] Decisions 1–3 wired to flags **and** `Engine.choicelog.record(...)`; ruins vision (`visionSeen`)
- [ ] Ship: unlock at 50%, 6-DoF flight, fuel, ship weapon, board/exit
- [x] Readiness (three-track formula + badge/panel)
- [ ] Summon at 100% → **`Engine.goto('hollowVerge')`** carrying appearance/weapon/flags
- [ ] Audio (SFX set + ambient wind + tension score + footsteps)
- [ ] Settings (difficulty/colorblind/graphics) via shared `settings:v1`
- [ ] Save/load (`save:v1` full shape) + autosave
