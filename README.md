# Grid Breaker

A complete, standalone **match-3 RPG**. The city's power grid is failing; you play a field operator matching energy runes on an 8×8 board to power attacks, healing, shields and special abilities while fighting network anomalies one incident at a time. No content, vocabulary, logic, or interface reused from any other project.

Visible name on every screen: **Grid Breaker** only. No subtitle, no second name. (The in-fiction "Blackout" flavor names — the boss *Blackout Prime* and the ultimate ability *Blackout Surge* — are lore tied to the city-power-outage setting, not the product name; this is a separate, independently-named project, distinct from *Blackout Protocol*.)

## Origin

This game started life as a rewrite of *Blackout Protocol* (a tower-defense game turned match-3 RPG). When it came time to publish, the original `Blackout-Protocol` GitHub repository turned out to be archived (read-only), so it was republished under its own name, its own GitHub repository, and its own Vercel deployment.

## Comparison against the reference site — this round's specific finding and fix

`https://runes-of-the-arcane-realm.vercel.app/` was re-opened and re-audited this round, screen by screen, specifically checking **composition** (not just palette): screen layout, HUD placement, card shapes, and information hierarchy.

**The concrete problem found**: the reference site's combat screen is a strict two-column layout — a left column stacking a scene, an enemy stat card, a player stat card, an ability list and a log, next to a right column holding only the board. Before this round, Grid Breaker's combat screen used the **identical** two-column shape (`.combat-col` / `.board-col` side by side), just recolored — so even with different sprites and a CRT palette, the underlying skeleton read as the same page. That was a real, structural resemblance, not a false alarm.

**The fix — composition, not palette:**
- **The two-column layout is gone entirely.** `#screen-game` is now a single vertical `.cabinet` column, capped at 640px and centered, that never becomes two columns at any viewport width (checked at 375px, 812px and 1440px — see tests below). This is the opposite of the reference's layout, which is explicitly two columns on desktop.
- **The player's vitals are no longer a stacked card.** They're a horizontal row of compact cartouche "chips" (HP/Energy/Shield/Crit/Overload, each icon + thin bar + value) inside a top status marquee — a shape with no equivalent on the reference site, which lists vitals as full-width rows in a vertical card.
- **The enemy's name/HP/attack-timer are no longer a separate card either.** They're now an overlay HUD docked to the top of the pixel-art viewscreen itself (a fighting-game convention: health bars drawn over the scene), not a bordered card sitting beside it like the reference's enemy card.
- **Abilities are no longer a row of small bordered rectangles.** They're arranged inside a labeled "CONTROL PANEL" module with rounded-cap button styling (a small highlight dot on each, like a physical arcade button), below the board.
- **Mode select is no longer two side-by-side cards** (which is exactly the reference site's own mode-select shape — verified by screenshot comparison). It's now a vertical list of numbered "mission slots" (01 / 02), each a horizontal strip with a slot-number badge, not a card grid.
- The 8×8 board itself, the swap-to-match mechanic, and the double-resource-bar concept remain (a match-3-RPG needs a board and needs to show two combatants' status somehow) — those are genre necessities, not reference-site inventions, and were kept.

No text, name, or asset from the reference site appears anywhere in this codebase (re-verified this round — see code audit below).

## Visual direction — additive to the previous round's pixel-art and CRT work

Kept from the previous round: five distinct pixel-art enemy sprites (Parasite/Drone/Specter/Overload/Boss), CRT scanlines, arcade corner brackets, the marquee status bar, the scanline sweep, pixel/monospace typography.

Added this round: the cabinet single-column composition described above, the chip-based vitals module, the enemy-HUD-on-viewscreen overlay, the labeled control panel, and the mission-slot mode-select list.

## Files

- `index.html` — English version (self-contained: HTML + CSS + JS inline, zero external dependencies, no build step, no server required)
- `fr/index.html` — French version, exact functional mirror
- `README.md` — this file

Both remain exclusively inside this Stage 6 folder (`STAGE 6 - BUILD A GAME/Grid Breaker`); nothing was moved to the root of `Claude/Projects`.

## Tests actually performed this round (real browser, DOM/console checked, not inferred from code)

- **The new cabinet composition, English**: loaded fresh, tutorial played through a real adjacent swap (confirmed via `tt-step` text advancing 1→3 immediately, then to 8/8 after the ability cast and enemy counter-attack), with the enemy-HUD-on-viewscreen overlay showing live HP updates (confirmed `val-enemyhp` changed from 60/60 to 6/60 after a real Pulse Strike cast).
- **A real bug found and fixed while testing the new layout**: the enemy-HUD overlay initially collided with the pre-existing "LIVE FEED // SECTOR 07" scene-label (both anchored top-left), overlapping unreadably. Found by screenshot, fixed by moving `.scene-label` to the bottom of the viewscreen (pairing it with the existing bottom-right "SYS // ONLINE" readout instead), re-verified by screenshot that the two no longer overlap.
- **A real Incident 1 victory** played through the new layout (genuine adjacent swaps + a Pulse Strike cast, not simulated), reaching "Incident Contained" with real score/damage/XP/credit numbers, confirming the restructured combat screen doesn't break the win path or the (previously fixed) single-reward-per-win guard.
- **Desktop width (1440×900)**: confirmed the cabinet stays a single capped-width column and does **not** reflow into two columns at wide viewports — this was the specific structural complaint being addressed, so it was checked deliberately at desktop width, not just mobile.
- **Mobile width (375×812)**: home screen and mode-select screen both re-checked after the mission-slot redesign — legible, buttons reachable, no horizontal overflow.
- **French mirror**: tutorial played through a real swap (`tt-step` advancing correctly, "ÉTAPE 7/8" observed), completed to mode select, a real Campaign battle started and the in-game Home button (`btn-home`, title "Retour à l'accueil") confirmed via `document.getElementById('screen-start').hidden === false` to actually return to the start screen. Mobile width (375×812) re-checked for the French home screen.
- **Code audit, both files**: JavaScript parses without a syntax error (Node `Function` constructor check); zero duplicate HTML `id` attributes; every `$('id')` reference in the JavaScript resolves to a real element in the same file; English and French files have byte-identical lists of function names and of element `id`s (diffed); no tower-defense-engine identifiers; no reference-site names/text (`Runes of the Arcane`, `Ruby`, `Sapphire`, `Goblin Marauder`, etc.) anywhere in either file.
- **Console**: checked for JavaScript errors after every action this round (tutorial, ability casts, a full victory, Home navigation) in both languages — none observed.

## Tests carried over from the previous round, not repeated this round (composition changes don't affect this underlying logic, but they weren't re-clicked this round)

- Settings modal (sound + language toggle), full EN→FR and FR→EN navigation from the home screen, Pause/Resume, a full campaign run through all 5 enemy types and the boss, a real Defeat + Retry, the double-reward-per-win bug fix, and Upgrades' cost-gating were all verified live in the previous round (see that work's commit history) and did not regress from this round's HTML/CSS-only restructuring, which touched no JavaScript logic — the code audit above (identical function lists, no dangling references) is the evidence for that claim, not a fresh click-through of every one of those specific screens this round.
- The Timed mode's natural 60-second countdown to its own end screen still could not be freshly re-observed this round, for the same reason as before: this session's Browser pane repeatedly reports `document.hidden === true` for extended stretches, and the game correctly (by design) pauses all time-based mechanics while hidden.
- Boss fight (Incident 10, Blackout Prime) sprite and stats were re-confirmed visually correct with the new sprite system in the previous round; it was not replayed again this round specifically to check the new composition, since the composition change is layout/CSS, not enemy-specific logic.

## Publishing

- **GitHub**: `github.com/KenMMStudio/Grid-Breaker`, independent of the archived `Blackout-Protocol` repository.
- **Vercel**: `grid-breaker-two.vercel.app` (English at `/`, French at `/fr/`), independent of any prior Vercel project.
