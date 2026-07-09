# Beyond the Hollow — Project Handoff Summary

*Prepared as a handoff package so a new Claude.ai Project can pick up this work with full context, without needing to re-explain anything.*

---

## 1. What This Game Is

**Title:** Beyond the Hollow
**Genre:** Narrative sci-fi survival/exploration, browser-based (Three.js), two connected worlds
**Tone:** Epic and hopeful, never grimdark — a found-family crew building something against the odds

**Premise:** The player commands a small expedition crew whose ship crashes on a hostile frontier world. They must survive, build a colony, and eventually earn passage to a second, much larger location — an ancient alien megastructure — by proving themselves worthy to its ancient AI custodian, **Ashketh**.

**Full story bible:** see the attached `beyond_the_hollow_design_doc.md`. That document covers the complete premise, both planets' design, the two companions (Reyes and Osei), the four big branching decisions, and Ashketh's full backstory. **Note:** that doc predates most of the actual build below — treat it as the narrative foundation, and this summary as the current implementation state layered on top of it.

---

## 2. Current Build State

### Planet A — "Kessa's Reach" (`world_prototype.html`, ~4,150 lines)

The first, fully-developed world. Built and polished across many sessions. Includes:

- **Exploration & movement** — full 3rd-person movement, day-night cycle, terrain with rolling geography, ruins cluster, alien flora
- **Character system** — full customization (size, build, suit/leg/skin color, headgear), carried forward into Planet B
- **Combat** — melee combat, weapon armory/progression system, enemy AI (wildlife + rival faction)
- **Colony building** — dual-resource system (Wood + Scrap Metal), build menu, structures
- **Companions** — Reyes (pilot/mechanic) and Osei (scientist) with full branching dialogue trees, plus Kade (recruited rival leader) with his own ongoing mechanical contribution to the colony
- **Rival faction encounter** — fight-or-truce decision point, shapes later dialogue and the ending
- **The ship** — Reyes repairs it once Readiness hits 50%; full 6-directional flight, ship-mounted weapon, fuel system that recharges at the colony beacon
- **Ashketh's presence** — fleeting vision moments on Planet A, then a full scripted **summoning sequence** once Readiness hits 100%, which transitions the player to Planet B (opens `planet_b_prototype.html`)
- **Visual pass** — full post-processing pipeline (bloom, ACES tone mapping, cinematic grade), procedural ground/sky textures, ambient particles, rim lighting
- **Audio** — procedural ambient score (tension-reactive), full SFX set, footsteps
- **Accessibility** — difficulty scaling, colorblind mode, graphics quality toggle (settings shared with Planet B)
- **Save system** — full save/load via browser storage, autosave
- **Polish systems** — death animation, talking/gesture animation during dialogue, cutscene camera system, settings panel
- **NEW: Dynamic ambient dialogue** — Reyes, Osei, and Kade's repeatable "loop" dialogue lines can be freshly generated live via the Claude API (falls back instantly and silently to hand-written variant lines if unavailable — see Section 4, this only works live inside Claude.ai's artifact preview)

### Planet B — "The Hollow Verge" (`planet_b_prototype.html`, ~2,000 lines)

Built later, brought up to parity with Planet A across a full audit pass. Includes:

- **The ring-world environment** — Architect megastructure aesthetic, cold mechanical lighting, structural pillars, debris
- **Two enemy types** — Guardian constructs (ranged, disciplined) and mutated creatures (fast melee), matching the design doc's "two-threat" structure
- **Three Threshold trials** (of the doc's planned 3–4):
  - **Trial of Strength** — 3-wave combat gauntlet
  - **Trial of Restraint** — reach an altar without striking back at provocateurs
  - **Trial of Cooperation** — a genuinely different mechanic: guide an ally construct (the "Warden Fragment") to hold one beacon while the player holds another simultaneously; attacking the ally fails the trial
  - All three retry-on-fail, no permanent lockout, per the design doc
- **Choice echo system** — Ashketh's dialogue at the start of each trial, and the final ending text, both genuinely reflect the player's actual Planet A decisions (rival faction outcome, Reyes' confession handling, Osei's discovery choice) — read live from Planet A's save data
- **The ending ("The Threshold")** — one ending sequence with tone that shifts across multiple tiers based on how many of the three tracked decisions leaned toward trust vs. self-reliance
- **Footholds** — a re-skinned, smaller-scale version of Planet A's base-building: a **Waypoint Beacon** (fast travel + stamina restore) and a **Repair Station** (passive HP regen), per the design doc's "footholds, waypoints, repair stations" framing. Simplified to instant-construct (no separate resource economy on this side)
- **Character/weapon carryover** — reads the same shared storage Planet A writes to, so the player's look and equipped weapon visually carry over
- **Visual pass** — brought to full parity with Planet A: same post-processing pipeline (tuned colder/bluer), efficient GPU point-sprite ambient particles (not individual meshes — this was an actual performance bug that got fixed)
- **Audio** — procedural ambient score (deliberately more dissonant/mechanical than Planet A's, to differentiate the megastructure's mood), full SFX set
- **Accessibility** — same three settings as Planet A, **sharing the same storage key** so choices carry across both worlds automatically
- **NEW: Dynamic Ashketh musings** — occasional live-generated idle lines while exploring outside trials, same fallback safety as Planet A's dynamic dialogue

---

## 3. Shared Technical Architecture (Important for Claude Code)

**The two files are NOT a unified codebase.** They are two large, independent HTML files that communicate *only* through shared browser storage keys:

| Storage key | Written by | Read by | Purpose |
|---|---|---|---|
| `character:config` | Planet A | Planet B | Character appearance carryover |
| `weapon:state` | Planet A | Planet B | Equipped weapon carryover |
| `settings:v1` | Both | Both | Shared accessibility settings |
| `save:v1` | Planet A | Both | Planet A's save data + story decision flags (also read by Planet B for choice-echo and ending) |
| `save:planetb:v1` | Planet B | Planet B | Planet B's own save data (position, trial progress, footholds) |

This works, but it's fragile — there's no real shared player/combat/dialogue system, no shared code, and no build pipeline. **This is the single biggest structural task for Claude Code**: merging both into one real project with shared systems, rather than two linked prototypes.

**Important caveat on the dynamic dialogue feature:** it calls `fetch("https://api.anthropic.com/v1/messages")` directly from the browser. This only succeeds because Claude.ai's artifact preview sandbox proxies that specific call without needing an API key. Once these files are opened standalone — downloaded, hosted elsewhere, or running inside a Claude Code dev environment — that proxy won't exist, and the fetch will fail. **This is handled gracefully already**: every dynamic-dialogue call has a timeout and try/catch that falls back to the hand-written static lines, so nothing breaks. But if you want the *live* generation to keep working after moving to Code, you'll need to wire up a real API key server-side there.

---

## 4. Key Decisions Locked In During This Build

- Three Threshold trials built (Strength, Restraint, Cooperation) — the design doc's "3–4" range, treated as complete
- Foothold/base-building on Planet B simplified to instant-construct, no separate resource economy (a deliberate scope decision, not an oversight)
- Ending tone driven by three decisions (rival path, Reyes' confession, Osei's discovery) — four tiers of outcome text
- Settings/accessibility intentionally shares one storage key across both planets rather than being configured twice

## 5. Known Gaps / Recommended Next Steps

In rough priority order:

1. **Merge Planet A and B into one unified codebase.** The clear first Claude Code task — shared player/combat/dialogue systems, proper file structure, real state management instead of storage-key handshakes.
2. **Choice log / visible consequence tracking** — discussed but not yet built. A UI element showing the player how their decisions are shaping the world, not just felt through dialogue/ending.
3. **Deeper accessibility** — remappable controls, subtitle/text-size options. Discussed but not yet built.
4. **Decide on the dynamic dialogue feature's future** — keep it Claude.ai-sandbox-only (fine, but limits where the game can be played), or wire a real API key once in Code (adds cost/complexity but keeps the feature working everywhere).
5. Optional/lower priority: a 4th Threshold trial, deeper Planet B base-building, environmental lore fragments.

---

## 6. Files in This Handoff

- `beyond_the_hollow_design_doc.md` — original story/design bible
- `world_prototype.html` — Planet A, current build
- `planet_b_prototype.html` — Planet B, current build
- This summary document
