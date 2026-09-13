# Grid Breaker

A complete, standalone **match-3 RPG**. The city's power grid is failing; you play a field operator matching energy runes on an 8×8 board to power attacks, healing, shields and special abilities while fighting network anomalies one incident at a time. No content, vocabulary, logic, or interface reused from any other project.

Visible name on every screen: **Grid Breaker** only. No subtitle, no second name. (The in-fiction "Blackout" flavor names — the boss *Blackout Prime* and the ultimate ability *Blackout Surge* — are lore tied to the city-power-outage setting, not the product name; this is a separate, independently-named project, distinct from *Blackout Protocol*.)

## Origin — why this is a new, separate project

This game started life as a rewrite of *Blackout Protocol* (a tower-defense game turned match-3 RPG). When it came time to publish, the original `Blackout-Protocol` GitHub repository turned out to be **archived (read-only)**, so the rewritten game could not be pushed there. Rather than fight the archive, this became its own project with its own name, its own GitHub repository, and its own Vercel deployment — a genuinely different game name from *Blackout Protocol*, as requested.

The reference site (a match-3 RPG with an 8×8 board, double combat HUD, ability bar and rune-role legend) was studied live in a browser before writing any code. Grid Breaker borrows the *functional shape* — board → resource generation → combat — but uses an original sci-fi "city blackout" setting, its own 7-symbol rune set and roles, its own ability names, its own enemy roster, and CSS-drawn glowing silhouettes instead of pixel-art fantasy sprites, so it is not a visual copy of that reference.

## Files

- `index.html` — English version (primary, self-contained: HTML + CSS + JS inline, zero external dependencies, no build step, no server required)
- `fr/index.html` — French version, exact functional mirror (same board, same enemies, same abilities, same progression, same screens — only the text is translated)
- `README.md` — this file

Open either file directly by double-clicking it in a browser, or serve the folder with any static file server. Both work offline.

## Core mechanics

- **Board**: 8×8 grid, 7 rune types (Pulse/orange, Energy/cyan, Repair/green, Surge/purple, Charge/yellow, Shield/white, Overload/red). Tap a rune, then tap an adjacent rune to swap; a swap that creates no line of 3+ reverts with an on-screen message. Matches of 3/4/5+ scale damage/effect ×1/×1.5/×2, and cascades add +0.2 per chain step.
- **Rune roles**: Pulse and Surge deal damage to the enemy; Energy fuels abilities; Repair heals; Shield grants a shield that absorbs incoming damage before HP; Charge fills a critical meter that doubles the next damage instance at 100%; Overload fills the ultimate meter but also deals small self-damage per tile matched (risk/reward).
- **Combat**: double HUD (enemy card + operator card) above the board, real-time enemy "attack in" timer independent of the board, 4 abilities (Pulse Strike, Grid Repair, Emergency Shield, Blackout Surge — the last requires a full Overload meter instead of Energy), keyboard shortcuts `1`–`4`, a live combat log.
- **Campaign mode ("Incident Response")**: successive battles, 4 rotating enemy types (Electric Parasite, Corrupted Drone, Network Specter, Unstable Overload) that scale in HP/damage with battle number, every 5th battle is a "major blackout event" boss (Blackout Prime) with a periodic overcharged attack. XP, levels and credits persist in `localStorage`; victory/defeat screens show real stats and route to Next Incident / Upgrades / Home or Retry / Upgrades / Home.
- **Timed mode ("Grid Overdrive")**: 60s (+8s per Overdrive Capacitor upgrade) score-attack variant on the same board/match engine, no enemy, combo multiplier tracked, best score saved.
- **Upgrades**: 5 permanent purchases (max HP, max Energy, match/ability damage %, max Shield, Overdrive duration), gated by credits, persisted.
- **Tutorial**: 8-step interactive, gated flow — forces a real adjacent swap, explains the resulting match/damage, explains energy generation, gates on actually pressing/tapping the Pulse Strike ability, explains the enemy counter-attack when it actually happens, then requires finishing a low-HP training target. Auto-launches on first Deploy; replayable anytime via "How to Play" without touching saved progress or real campaign state.

## Tests actually performed (real browser, carried over from the Blackout Protocol rewrite this game started from, then re-verified after rebranding)

- **Tutorial, English and French**: played step-by-step to completion — real adjacent swap, real match, real Pulse Strike cast, real enemy counter-attack observed, real training-target kill, landed cleanly on mode select with the tutorial-seen flag set and zero effect on real campaign/credit state.
- **Campaign combat, English**: a real defeat (Operator Down screen, correct stats, Retry restarted the same incident at full HP) and a real victory (Incident Contained screen with real score/damage/XP/credits), followed by a real transition to Incident 2 with a correctly-scaled different enemy (Corrupted Drone, 109 HP) and persisted level/XP.
- **Campaign combat, French**: tutorial played independently, then a real victory ("Incident maîtrisé") and a real defeat on Incident 2 ("Opérateur hors service") with correct localized stats, confirming the French build is a true functional mirror, not just translated static text.
- **Abilities, pause, invalid-swap rejection, timed mode's 60s countdown and its zero-score end screen, and the upgrades screen's cost-gating** were all exercised live — see the full test log in this project's prior name (kept in the `Blackout Protocol` project history) for the exact numbers observed.
- **Responsive layout**: desktop wide, 768×1024 tablet, 812×375 phone landscape, 375×812 phone portrait all verified legible and reachable, switching from a two-column desktop composition to a single stacked column under 900px.
- **Console**: checked for JavaScript errors after every major action across both languages — none observed.
- **After rebranding to Grid Breaker**: verified the title, on-screen brand mark ("GRID BREAKER"), and `localStorage` keys (`gb_save_v1`, `gb_tutorial_seen_v1` / `_fr`) all changed correctly and no "Blackout Protocol" text remains anywhere except the in-fiction boss/ability flavor names explained above.

## Tests NOT performed — stated honestly

- **Real touch input on physical hardware** was not tested.
- **A full campaign run to a boss battle (Incident 5)** was not played to a live conclusion — the boss-scaling code path is implemented and code-reviewed but not observed live.
- **The Timed mode was not driven to a positive nonzero score** in the session that produced this code — the zero-score end-screen path was verified, and the identical scoring code was verified indirectly via real Campaign-mode damage numbers.
- **Purchasing an upgrade** was not completed live (credits earned never reached the cheapest cost); the gating logic was verified, the purchase flow only by code review.
- **This specific renamed copy was not replayed end-to-end after rebranding** — only the branding/text change itself was verified (title, header, save keys, no leftover old name); the underlying game logic is byte-identical to the already-tested `Blackout Protocol` rewrite, so no functional regression is expected, but it has not been independently re-proven under the new name.
- **No sound**: there are no audio assets; the speaker icon is a visual toggle only.
- **No video** has been created.

## Publishing — new, independent GitHub repository and Vercel project

- **GitHub**: created fresh for this project (see the repository's own URL on GitHub for the live link — normalized from "Grid Breaker" per GitHub's naming rules).
- **Vercel**: created fresh, connected only to this new GitHub repository, not to any prior Vercel project.

Neither is connected to `Blackout-Protocol` (archived) or to any other prior project.
