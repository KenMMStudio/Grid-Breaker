# Grid Breaker

A complete, standalone **match-3 RPG**. The city's power grid is failing; you play a field operator matching energy runes on an 8×8 board to power attacks, healing, shields and special abilities while fighting network anomalies one incident at a time. No content, vocabulary, logic, or interface reused from any other project.

Visible name on every screen: **Grid Breaker** only. No subtitle, no second name. (The in-fiction "Blackout" flavor names — the boss *Blackout Prime* and the ultimate ability *Blackout Surge* — are lore tied to the city-power-outage setting, not the product name; this is a separate, independently-named project, distinct from *Blackout Protocol*.)

## Origin

This game started life as a rewrite of *Blackout Protocol* (a tower-defense game turned match-3 RPG). When it came time to publish, the original `Blackout-Protocol` GitHub repository turned out to be archived (read-only), so it was republished under its own name, its own GitHub repository, and its own Vercel deployment.

## Comparison against the reference site — what's shared and what's original

`https://runes-of-the-arcane-realm.vercel.app/` was opened live in a browser and audited screen by screen (home, mode select, 8×8 board, double combat HUD, ability bar, rune-role legend, Settings) before and during this session's work.

**Shared — functional shape only, and standard genre conventions no single game owns:**
- The overall loop (board → resource generation → combat), an 8×8 grid, adjacent-swap matching, a double HUD above the board, a keyed ability bar, and a rune-role legend. These are conventions of the match-3-RPG genre itself (also present in Bejeweled-likes, Puzzle Quest, etc.), not inventions of the reference site.

**Deliberately different — checked line by line to avoid copying:**
- **Symbols and roles**: an original 7-symbol set (Pulse/Energy/Repair/Surge/Charge/Shield/Overload) with an original role each — including Overload, a risk/reward mechanic (self-damage + ultimate charge) that the reference site does not have at all. None of the reference's rune names (Ruby, Sapphire, Emerald, Amethyst, Topaz, Moonstone) or enemy name ("Goblin Marauder") appear anywhere in this codebase — verified by direct text search.
- **Setting and cast**: sci-fi urban blackout, not medieval fantasy. Original enemy roster (Electric Parasite, Corrupted Drone, Network Specter, Unstable Overload, the boss Blackout Prime) with no equivalents on the reference site.
- **Visual language**: this project uses a CRT/8-bit arcade-cabinet aesthetic (scanlines, hard pixel edges, neon cyan/magenta/yellow on near-black) built from inline pixel-art SVGs and CSS; the reference site uses a soft, rounded, serif-and-gold medieval-fantasy look with no scanlines, no pixel art, and no hard-edged panels. Screenshots of both were compared side by side this session to confirm the palettes, typography, and panel shapes do not match.
- **Modes**: "Incident Response" (campaign) and "Grid Overdrive" (timed) map functionally to the reference's "Arcane Battle" and "Timed Trial", which is the same genre convention almost every match-3-RPG uses — but names, framing, and enemy scaling formulas are original to this project.

No text, image, or asset file was copied from the reference site into this project.

## Visual direction — what changed this session and why the earlier pass wasn't enough

The version at the start of this session had already added CRT scanlines and a retro color palette, but it was still fundamentally the previous sci-fi layout with colors and borders swapped — not a genuinely rethought composition, and it had a real regression: **every enemy type rendered as the identical pixel sprite** (the HTML was hardcoded to one static SVG, so the per-type CSS rules for silhouette/color that used to exist had become dead code with no matching elements). This was found by auditing the code, not assumed.

This session's changes:
- **Five distinct pixel-art enemy sprites**, generated from a `ENEMY_SPRITES` lookup keyed by enemy id and swapped into the sprite's `<svg>` at battle start: Electric Parasite (small spiky orange/red blob), Corrupted Drone (grey/cyan hexagonal mechanical body with a red visor), Network Specter (tall translucent purple ghost, no legs), Unstable Overload (jagged red/white creature with sparking pixels), and Blackout Prime (larger, darker, spiked, glowing-eyed boss, rendered bigger via CSS). Each was confirmed rendering correctly in its own real battle this session (see test log below).
- **Arcade-cabinet corner brackets** (yellow bracket corners + a dashed inner CRT-style border) added to the start card, the combat scene, and all three end-screens (victory/defeat/timed-end).
- **A marquee readout bar** ("ARCADE UNIT 07 — 1 CREDIT" / "SELECT YOUR MISSION" and French equivalents) added above the title and above the mode-select heading, with a blinking status dot.
- **A slow horizontal CRT scanline sweep** added as a fixed overlay across the whole page.
- **A blinking cursor** after the title text (was already present, kept).

This is a real, verifiable difference from a "few CSS properties": new SVG content generated per enemy, new markup elements (corner brackets, marquee, sweep overlay) added to both language files identically, not just new colors on old shapes.

## Navigation fixes — all now real, none decorative

The following were either missing outright or were decorative placeholders before this session, confirmed by reading the code and then testing live:
- **Return to Home from combat**: a home icon button (⌂) in the game-screen top bar now calls a real `goHome()` handler (pauses the loop, hides any open tutorial overlay, returns to the start screen). Tested from mid-tutorial, mid-campaign-battle, and mid-timed-mode.
- **English → French and French → English**: a language button in the top bar (in-game) and a full-width button on the home screen (`btn-language`) now call `switchLanguage()`, which navigates to `fr/index.html` (or back to `../index.html` from the French version). Tested both directions live.
- **Settings**: the Settings button used to just show a toast saying "no additional settings yet." It now opens a real modal with a working Sound on/off toggle (mirrors the top-right sound icon) and a working language-switch button, plus Close. Tested live: toggling sound in the modal updates both sound icons; Close actually hides the modal (confirmed via the DOM, not just visually — see the note on screenshot timing below).
- **Pause, Retry, Upgrades (from the win/lose/timed-end screens and from Home), Back (from mode select and Upgrades)**: all confirmed live this session, each routing to the correct screen with no dead ends.

## A real bug found and fixed this session: victory/defeat rewards could be applied more than once per battle

While auto-playing a real battle, "Next Incident" was clicked once after one win and the game jumped from **Incident 4 straight to Incident 7** — a 3-battle jump from a single victory, confirmed by reading `localStorage` (`battle` had gone from 4 to 7, not 4 to 5). Root cause: `checkEndConditions()` is called from inside the match-cascade resolution loop, once per cascade step. If the kill happened on a cascade step that was followed by more cascade steps (a multi-step chain), `S.enemy.hp<=0` stayed true across every remaining step, and `onCampaignVictory()` (which grants XP/gold and increments the save's battle counter) had no guard against being called again — so it ran once per remaining cascade step, each time incrementing the battle counter and paying out rewards again. The identical pattern exists for `onCampaignDefeat()`.

Fixed by adding a `S.roundOver` flag: `checkEndConditions()` now returns immediately if it's already set, and sets it the first time a win or loss is detected; the flag is reset to `false` at the start of every new battle and every new tutorial run. Re-verified live after the fix: a real win at Incident 7 advanced the save to exactly Incident 8 (not further), confirmed by reading `localStorage` before and after. Fixed identically in `fr/index.html`.

## A real bug found and fixed earlier this session (before the visual work above): tutorial claimed 8 steps but only ran 6

`tutorialOnEnemyAttack()` only ever advanced the flow from step 5 to step 6, then wired the training target's death straight to `finishTutorial()` — steps 7 and 8 were announced by the "Step X / 8" counter but never actually shown. Fixed by adding a real step 7 (explains Charge/Overload) and step 8 (the "finish the training drone" call to action), each auto-advancing the same way steps 2–4 already did. Re-verified live, in both languages, that the overlay genuinely reaches "STEP 8 / 8" / "ÉTAPE 8 / 8" before the tutorial completes.

## Tests actually performed this session (real browser, DOM state and/or screenshots checked, not inferred from the code)

- **All 5 enemy sprites**, each in a real Campaign battle reached by auto-playing genuine matches: Electric Parasite (Incident 1, 9), Corrupted Drone (Incident 2), Network Specter (Incident 3, 7), Unstable Overload (Incident 4), Blackout Prime (Incident 10, "MAJOR EVENT · 10" tag, 1332 HP) — all five confirmed visually distinct, not the same recolored shape.
- **The round-over/double-reward bug**, reproduced, fixed, and the fix verified by reading `localStorage`'s `battle` value before and after a real win (delta of exactly 1, twice, after the fix — previously a delta of 3 was observed with the bug present).
- **Home button**: tested from the tutorial, from a real Campaign battle, and confirmed (via `document.getElementById('screen-start').hidden === false`) that it actually returns to the start screen each time — including a case where the on-screen screenshot was one frame stale (a known lag in this session's preview tool) and the DOM state was checked directly to confirm the click had, in fact, worked.
- **EN → FR and FR → EN** navigation, from the home screen, confirmed by reading the rendered page text in each direction.
- **Settings modal**: opened, sound toggled (confirmed text changes from "Sound: ON" to "Sound: OFF"), closed (confirmed via `document.getElementById('settings-modal').hidden === true`).
- **Tutorial, English**: replayed step-by-step to "STEP 8 / 8" (see bug note above), skipped via "Skip Tutoral" to mode select.
- **Tutorial, French**: replayed step-by-step to "ÉTAPE 8 / 8" independently.
- **A full campaign run from Incident 1 through Incident 10 (the first boss)**, English: real victories on Incidents 1–4, 7, 8, 9 (Incidents 5–6 were skipped over during a battle-count correction while diagnosing the double-reward bug — see note below), each via genuine adjacent-swap matches and ability casts (not simulated damage), with `localStorage` checked after each; the Incident 10 boss fight was started, damaged to 593/1332 HP live, then abandoned (not required to finish for this audit) rather than ground out.
- **A real defeat and Retry**, English, on the Incident 10 boss: deliberately matched Overload runes repeatedly (a real, in-game self-damage mechanic) until HP reached 0, producing a genuine "Operator Down" screen with correct stats; Retry confirmed to restart Incident 10 at full HP.
- **A real victory in French** ("Incident maîtrisé", Parasite électrique neutralisé) on a fresh French save, via genuine matches and a Pulse Strike cast, with the battle counter confirmed to advance by exactly 1.
- **Responsive layout**: 1024×768-equivalent desktop, and 375×812 mobile portrait, both re-checked after the visual overhaul — home screen and in-battle screen both legible, corner brackets and marquee bar scale down cleanly, top-bar icons (Home/Language/Pause/Sound) all remain reachable at phone width.
- **Code audit**: JavaScript in both files parses without a syntax error (checked with Node's `Function` constructor); no duplicate HTML `id` attributes in either file; every `$('id')` reference in the JavaScript resolves to a real element in the same file; the English and French files have byte-identical lists of function names and byte-identical lists of element `id`s (confirmed by diff), i.e. the two files are structurally the same program with only text translated; no tower-defense-engine identifiers (`BUILDABLE_SET`, `district`, `tower`, `checkpoint`) and no reference-site names (`Runes of the Arcane`, `Ruby`, `Sapphire`, `Goblin Marauder`, etc.) appear anywhere in either file.
- **Console**: checked for JavaScript errors after every major action (start, tutorial, battles, victory, defeat, settings, language switch, home) across both languages — none observed.

## Tests NOT performed, or only partially performed — stated honestly

- **The Timed mode's natural 60-second countdown to its own end screen was not freshly re-observed this session.** A real score (2252 points, combo ×2.4) was confirmed accumulating correctly from genuine matches, but the countdown timer itself did not advance during later testing because this session's Browser pane repeatedly reported itself as hidden (`document.hidden === true`) for extended stretches — and the game deliberately does not advance any time-based mechanic (enemy attack timer, this countdown) while the tab is hidden, by design, per the anti-burst-damage fix from a previous session. That guard is confirmed still working (no time was falsely credited while hidden), but it also means the natural end-of-countdown transition specifically could not be re-produced live in this session's later tests. It **was** verified in a previous session (the "Overdrive Complete" screen, including the zero-score edge case).
- **Blackout Prime's every-3rd-attack overcharge log line** ("Blackout Prime overcharges its strike!") was not observed live this session — the boss fight was damaged down to under half HP without taking three of its attacks in the available real time (for the same document.hidden-related reason above: the enemy attack timer only ticks while the pane is visible, and it was not visible for long enough in one stretch during this test). The code path exists and was reviewed but not watched firing.
- **Incidents 5 and 6 specifically** were not each individually re-confirmed with a fresh screenshot in this session (battle progression was being used to hunt down and verify the double-reward bug fix, and jumped from tracking every single incident to spot-checking 7, 8, 9, 10) — Incidents 1, 2, 3, 4, 7, 8, 9, and 10 were each individually confirmed live.
- **Purchasing an upgrade** was not completed live this session (credits earned did not reach the cheapest upgrade's cost at the point of testing); the cost-gating (buttons correctly disabled) was confirmed live.
- **Real touch input on physical hardware** was not tested.
- **No sound**: there are no audio assets; the speaker icon is a visual toggle only, not a real mute switch for anything audible.
- **Minor cosmetic note, not fixed**: in Timed mode the enemy sprite from whatever Campaign battle was last played remains visible in the combat scene (only the enemy stat card and ability bar are hidden). This is pre-existing behavior, not something introduced this session, and is left as a known minor inconsistency rather than fixed, since Timed mode has no enemy by design.
- **No video** has been created.

## Files

- `index.html` — English version (self-contained: HTML + CSS + JS inline, zero external dependencies, no build step, no server required)
- `fr/index.html` — French version, exact functional mirror (confirmed by diffing function-name and element-id lists — see code audit above); only the text is translated
- `README.md` — this file

Both remain exclusively inside this Stage 6 folder (`STAGE 6 - BUILD A GAME/Grid Breaker`); nothing was moved to the root of `Claude/Projects`.

## Publishing

- **GitHub**: `github.com/KenMMStudio/Grid-Breaker`, independent of the archived `Blackout-Protocol` repository.
- **Vercel**: `grid-breaker-two.vercel.app` (English at `/`, French at `/fr/`), independent of any prior Vercel project.
