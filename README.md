# Grid Breaker

## The previous round's "fix" was incomplete — caught by the user with a real screenshot, fixed properly this round

The previous round raised `.cabinet`'s cap from 640px to `min(900px,94vw)` and the board from 440px to 600px, verified there was no horizontal overflow, and declared it fixed. That check was too narrow: at real desktop widths (confirmed with a live screenshot at ~2000px), the interface was correctly centered and technically not overflowing, but it still sat as a small, fixed-width island with hundreds of pixels of dead space on both sides — and the topbar (capped at a leftover, unrelated `1180px`) didn't even share the same width as the `.cabinet` below it, so the whole screen looked structurally mismatched, not just "a bit small." Raising a hard pixel cap by a few hundred pixels was not the same thing as making the layout adapt to the screen, which is what was actually asked for.

**The real fix this round:**

```css
.topbar{max-width:min(1200px,92vw);}   /* was a hardcoded 1180px, independent of the cabinet */
.cabinet{max-width:min(1200px,92vw);}  /* was min(900px,94vw) */
#board{width:min(88vw,760px);height:min(88vw,760px);}  /* was min(88vw,600px) */
```

`.topbar` and `.cabinet` now share the exact same width formula, so the top bar and the game panels line up as one coherent column instead of two mismatched widths — confirmed by `getBoundingClientRect()` at 2000px width: both measure exactly 1200×… at the same left/right edges. The board grew again, to 760px. This is still a deliberate design choice (a centered single-column "cabinet," not a full-bleed layout — consistent with the arcade-cabinet aesthetic), but the ceiling is now high enough that at common desktop sizes (1440–1920px) the interface visibly fills most of the screen instead of reading as a small floating box.

## A real viewport/layout bug found by a second review and fixed this round

A prior round's report claimed the responsive layout was "fixed" without providing screenshots that actually proved it. A closer look at the CSS confirmed three real, related problems:

```css
.cabinet{max-width:640px;}                              /* far too narrow on desktop/tablet */
#board{width:min(88vw,440px);height:min(88vw,440px);}   /* board capped small even with room to grow */
```

On desktop and tablet widths, the whole game was squeezed into a narrow 640px column with large empty gutters on both sides, and the board never grew past 440px even at 1920px wide. There was no rule re-expanding the cabinet or the board for wider viewports.

**The fix widens the caps instead of patching around them:**

```css
.cabinet{max-width:min(900px,94vw);width:100%;margin:0 auto;...}
#board{width:min(88vw,600px);height:min(88vw,600px);...}
@media (min-width:900px){
  .viewscreen{height:290px;}
  .control-panel .ability{flex:0 1 130px;}
}
```

The cabinet now scales up to 900px (94vw on anything narrower) instead of stopping at 640px, the board scales up to 600px instead of 440px, and a new `min-width:900px` rule gives the viewscreen and ability buttons a bit more breathing room once the wider cabinet is in effect. Nothing about the single-column composition changed — only the numeric caps.

**Verified by real screenshots and `getBoundingClientRect()`/`scrollWidth` measurements, in both languages, at all 7 requested sizes:**

| Size | scrollWidth vs clientWidth | Cabinet / board |
|---|---|---|
| 1920×1080 (desktop) | equal, no overflow | 900×~1301 cabinet, 600×600 board |
| 1440×900 (desktop) | equal, no overflow | 900×~1301 cabinet, 600×600 board |
| 1024×768 (tablet) | equal, no overflow | 900×~1301 cabinet, 600×600 board |
| 768×1024 (tablet) | equal, no overflow | ~722×~1252 cabinet, 600×600 board |
| 390×844 (mobile) | equal, no overflow | 366×~1256 cabinet, 343×343 board |
| 375×812 (mobile) | equal, no overflow | 351×~1242 cabinet, 330×330 board |
| 812×375 (mobile landscape) | equal, no overflow | ~763×~1252 cabinet, 600×600 board |

At every size, `document.documentElement.scrollWidth === clientWidth` (zero horizontal overflow), and the board/scene/stats/abilities/legend/log all remain reachable through ordinary vertical scroll — no content is trapped off-screen. On desktop/tablet the board is now visibly much larger and the huge empty side gutters are gone; on phone widths the board still scales down via `min(88vw, 600px)` exactly as before.

**A tooling artifact hit during this round's testing, worth recording honestly:** the browser-preview tool's screenshot capture became unreliable (frozen/incorrectly-scaled frames) on a single tab that had been resized and navigated many times over the session. Cross-checking against `getBoundingClientRect()`/`scrollWidth` JS measurements proved the actual page layout was correct even when the screenshot looked wrong. The fix was to open a fresh browser tab for screenshots going forward — this is a limitation of the testing tool used this session, not a bug in the game.

## Full functional checklist replayed this round, both languages, real clicks/waits (not code review)

- **Home, mode select**: loaded fresh in both languages, mode-select cards still render correctly (badge/title/description/button in one row — the previous round's CSS-specificity fix was unaffected by this round's changes).
- **Settings modal + Sound ON/OFF**: opened mid-battle, sound toggled (🔊→🔇), confirmed via the settings card's own "Son : Activé/Désactivé" text, closed and confirmed `settings-modal.hidden === true`.
- **EN → FR and FR → EN**: switched from the in-game settings modal, confirmed by reading the resulting page's own text after navigation.
- **Full 8-step tutorial, French**: played with genuine adjacent-cell swaps (found programmatically from the live board state, not hardcoded coordinates) and a genuine ability cast (`btn-1`/Frappe d'impulsion), confirmed by reading `tt-step`'s text progressing to a real board victory landing cleanly on mode select.
- **Home-from-combat**: confirmed via `screen-start.hidden === false` after clicking Home mid-battle.
- **Pause/Resume**: confirmed by reading the enemy attack timer (`val-enemytimer`) frozen across a 3s wait while paused, then advancing again after Resume.
- **A full Campaign run to a real Victory, both languages**: real adjacent-swap matches (computed live from the board's actual rune layout, not scripted moves) played until enemy HP reached 0 — "Incident Maîtrisé" / "Incident Contained" shown with real, non-placeholder score/damage/XP/credit numbers in both languages.
- **Upgrades screen**: opened with real (low, ~25cr) credits — all upgrades correctly shown as disabled/unaffordable at that credit level, matching the cost-gating logic, in both languages.
- **A real Defeat + Retry, both languages**: let the enemy's attack timer run down repeatedly without healing until HP reached exactly 0 — "Opérateur Hors Service" / "Operator Down" shown with correct stats, Retry confirmed to restart the same Incident at full HP.
- **Grid Overdrive (Timed mode)**: started, played real matches (score and combo confirmed updating live) in both languages; the French run was watched to its own natural end ("Surcharge Terminée", real nonzero score of 120 and a new-best-score flag, credits +8); the English run was spot-checked for live scoring (score 60 after 3 real matches) rather than replaying the full 60s countdown a second time, since the identical JS timer/scoring path was already watched to completion in French this round.

## Comparison against the reference site — unchanged from previous rounds

`https://runes-of-the-arcane-realm.vercel.app/`'s two-column composition was already replaced with a single vertical `.cabinet` column in a previous round; this round's fix only changed numeric CSS caps (`.cabinet` max-width, `#board` width/height) and added one `min-width:900px` media rule — it did not reintroduce a two-column layout or touch any other composition element.

A complete, standalone **match-3 RPG**. The city's power grid is failing; you play a field operator matching energy runes on an 8×8 board to power attacks, healing, shields and special abilities while fighting network anomalies one incident at a time. No content, vocabulary, logic, or interface reused from any other project.

Visible name on every screen: **Grid Breaker** only. No subtitle, no second name. (The in-fiction "Blackout" flavor names — the boss *Blackout Prime* and the ultimate ability *Blackout Surge* — are lore tied to the city-power-outage setting, not the product name; this is a separate, independently-named project, distinct from *Blackout Protocol*.)

## Origin

This game started life as a rewrite of *Blackout Protocol* (a tower-defense game turned match-3 RPG). When it came time to publish, the original `Blackout-Protocol` GitHub repository turned out to be archived (read-only), so it was republished under its own name, its own GitHub repository, and its own Vercel deployment.

## A real, blocking bug found by a second reviewer and fixed this round

A prior round's report declared the mode-select screen validated based on screenshots that, on closer inspection, showed the card text rendering one word per line and the button overlapping the content. A second review of the actual CSS found the exact cause: two rules targeting the same selector, `.mode-card .btn`, with the same specificity —

```css
.mode-card .btn{width:auto;margin-top:0;flex-shrink:0;}   /* added when the card became a flex row */
...
.mode-card .btn{width:100%;margin-top:14px;}              /* leftover from the old vertical-card design, never removed */
```

Because the second (leftover) rule came later in the stylesheet, it won the cascade unconditionally — forcing the button to 100% width inside a flex row on every screen size, which starved `.mode-body` of space and wrapped its text into a single narrow column. A second, related leftover (`.mode-card p{min-height:60px}`, also from the old vertical-card design) was fighting an already-added `min-height:0` reset on the same selector.

**The fix removes the conflicting rules instead of patching around them**: the obsolete unconditional `width:100%` and the obsolete `min-height:60px` declarations are deleted; the single remaining base rule keeps the button at its natural width (`flex:0 0 auto`, `white-space:nowrap`) inside the row; the *only* other `.mode-card .btn` rule left in the stylesheet is the intentional one inside `@media (max-width:560px)`, which stacks the button full-width below the wrapped text on narrow phones — that one was always correct and untouched.

**Verified by real screenshot, not by re-reading the CSS**: captured at 1440×900 (button stays in its own box at the end of the row, paragraph text reads normally) and at 375×812 (badge/text/button stack vertically, full-width button, still normal horizontal text) — see the two screenshots taken this round. Fixed identically in `fr/index.html` (same two leftover rules existed there too, confirmed by grep before fixing).

## Comparison against the reference site — carried over from the previous round, not repeated from scratch

`https://runes-of-the-arcane-realm.vercel.app/` composition was already audited and diverged from in the previous round: the combat screen's two-column skeleton (scene+cards+abilities left, board right — identical in shape to the reference site) was replaced with a single vertical `.cabinet` column that never splits into two columns at any width, player vitals moved from a stacked card to horizontal chips, the enemy's stats moved from a side card to a HUD overlaid on the scene itself, abilities moved into a labeled "control panel" module, and mode-select moved from two side-by-side cards (the reference site's own shape) to a vertical list of numbered mission slots. This round's fix was CSS-only (two conflicting declarations) and did not touch that composition — re-confirmed by the screenshots above, which show the same single-column cabinet, chip-based vitals, and mission-slot list as before, just with the text/button bug corrected.

## Tests actually performed this round (real browser, DOM/console checked, screenshots taken)

- **The mode-card bug itself**: confirmed present before the fix (grep showing the two conflicting rules), fixed, then confirmed absent by screenshot at both 1440×900 and 375×812 — button in its own box, paragraph text reading normally, no letter-by-letter or word-by-word vertical text, in **both** `index.html` and `fr/index.html`.
- **Full code audit, both files**: JavaScript parses without a syntax error; zero duplicate HTML `id` attributes; every `$('id')` reference resolves to a real element; every `btn-*` id has exactly one `addEventListener` call; English and French have byte-identical function-name and element-id lists; no tower-defense-engine identifiers or reference-site names/text anywhere in either file.
- **Home screen**: loaded fresh, no console errors, all stats/buttons present.
- **Settings modal**: opened from the in-game top bar mid-battle (`btn-settings`), Sound toggled (confirmed text change "Sound: ON" → "Sound: OFF"), Close confirmed via `settings-modal.hidden === true`.
- **EN → FR and FR → EN**, twice each, from two different contexts: from the home screen and from the in-game top-bar language button mid-battle — each confirmed by reading the resulting page's own text/URL after navigation, not assumed.
- **Home button from mid-combat**, in both English and French, confirmed via `screen-start.hidden === false` after the click, including once mid-tutorial (before completion) in French.
- **Pause / Resume**: confirmed by reading the enemy attack-timer value before and after a real 2-second wait while paused (unchanged) and while resumed (decremented).
- **Tutorial, 8 real steps, both languages**: a genuine adjacent swap performed via real cell clicks (not a simulated match), progression confirmed by reading `tt-step`'s text through "Step 8 / 8" / "Étape 8 / 8" after a real Pulse Strike cast and a real enemy counter-attack.
- **A full Campaign run, English**: three consecutive real victories (Incident 1 → 2 → 3, including a level-up on the third), each via genuine adjacent-swap matches and ability casts, each confirmed on the "Incident Contained" screen with real score/damage/XP/credit numbers and no console errors.
- **A real Upgrade purchase — the first time this has actually been completed and confirmed in this project's history, not just checked as disabled**: after earning 55 credits, "Reinforced Frame" (40cr) was purchased live; confirmed via `localStorage`'s `up.hp` going from `0` to `1`, credits dropping from 55 to 15 on screen, and the next tier's cost correctly recalculating to 70cr.
- **Grid Overdrive (timed mode) run to its own natural end — the first time the real 60-second countdown has been observed completing in this project's history**, rather than only the zero-score edge case from a previous round: the countdown was watched decrementing in real time (60s → 7s → 0), ending on "Overdrive Complete" / "New best score!" with a genuine nonzero score (1380) and combo (×1.4), no console errors.
- **A real Defeat + Retry**: deliberately matched Overload/Charge/Shield runes (avoiding damage and healing types) until HP reached exactly 0 on Incident 4, producing "Operator Down" with correct stats (Incident Reached 4, Damage Dealt 139, Credits +14); Retry confirmed to restart Incident 4 at full HP (125/125).
- **Anti-double-reward guard**: re-checked across all of the above real victories — each one advanced the save's battle counter by exactly one and produced one XP/credit payout, with no repeat of the earlier multi-payout bug.
- **Five distinct enemy sprites**: Electric Parasite, Corrupted Drone and Network Specter were each seen live this round (Incidents 1–3); Unstable Overload and Blackout Prime were not re-encountered this specific round (the campaign run this round didn't reach Incident 5+) but were unaffected by this round's CSS-only fix and were already confirmed distinct in the previous round.
- **Desktop (1440×900) and mobile (375×812)**: both re-checked this round specifically for the fixed screen (mode select) and additionally for the home screen and in-combat screen at mobile width — all legible, no overflow, no letter-by-letter text, all top-bar icons (Home/Language/Pause/Sound) reachable.

## Tests NOT re-performed this round (unaffected by a CSS-only fix, verified in a previous round)

- Boss fight (Incident 10, Blackout Prime) and Incidents 5–9 specifically were not replayed this round.
- Victory/Defeat/Upgrades/Settings screens' pixel-perfect layout at every one of the previously-checked sizes was not re-screenshotted beyond the desktop+mobile checks above, since this round's fix touched only `.mode-card` and `.mode-body` rules, confirmed by diff not to affect any other selector.
- Real touch input on physical hardware was not tested.

## Tests NOT (fully) performed in this latest round (viewport/layout fix)

- **Grid Overdrive's 60s countdown was watched to its own natural end in French only** this round; the English run was confirmed live (real score/combo updating from real matches) but not replayed for the full 60 seconds a second time, since the timer/scoring code is byte-identical between the two files and was untouched by this round's CSS-only changes.
- **Boss fight (Incident 10) and Incidents 3–9** were not replayed this round in either language — only Incidents 1–2 were driven to real conclusions.
- **A real Upgrade purchase** was not completed this round — credits earned (~25) never reached the cheapest upgrade's cost (40cr), so only the disabled/unaffordable gating was re-confirmed live, not a completed purchase.
- **Real touch input on physical hardware** was not tested.

## Files

- `index.html` — English version (self-contained: HTML + CSS + JS inline, zero external dependencies, no build step, no server required)
- `fr/index.html` — French version, exact functional mirror
- `README.md` — this file

Both remain exclusively inside this Stage 6 folder (`STAGE 6 - BUILD A GAME/Grid Breaker`); nothing was moved to the root of `Claude/Projects`.

## Publishing

- **GitHub**: `github.com/KenMMStudio/Grid-Breaker`, independent of the archived `Blackout-Protocol` repository.
- **Vercel**: `grid-breaker-two.vercel.app` (English at `/`, French at `/fr/`), independent of any prior Vercel project.
