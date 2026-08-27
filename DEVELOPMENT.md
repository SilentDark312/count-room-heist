# Development notes — Count Room: Heist

This repo started as a full copy of [SilentDark312/count-room](https://github.com/SilentDark312/count-room)
at the commit it was forked from. Everything about the blackjack engine itself —
card/shoe logic, the rules-aware strategy functions, Illustrious 18 deviations,
Free Play / Challenge / Count Drill, the bankroll risk simulator, the CI pipelines —
is unchanged and undocumented here on purpose. **Read that repo's `DEVELOPMENT.md`
first** for all of that; this file only covers what's different in this fork: the
Heist (adventure) layer.

## What's actually new here

Everything lives in the same single `index.html`, in one new section clearly marked
`HEIST MODE (adventure layer)`, plus a 5th tab (`data-tab="adventure"`, labeled
"Heist" in the UI) and a small adventure banner injected into the top of the existing
Play tab.

- **`ADVENTURE_STOPS`** — the only place "content" lives, in order: The Rusty Nickel,
  The Silver Spur Saloon, The Riviera Room, The Vault (finale). Each stop is
  `{id, name, blurb, numDecks, stake, goal, heatLimit, palette, introStory, winStory,
  loseStory}`, where `introStory`/`winStory`/`loseStory` are arrays of `{who, text}`
  beats. **Adding a new stop is just appending an entry to this array** — locking is
  computed, not authored (see `isStopUnlocked` below), so a new entry is automatically
  locked until the one before it is cleared. No other code should need to change for
  an ordinary content addition; do re-run the balance simulation, though (see below).
- **`isStopUnlocked(idx)`** — stop 0 is always unlocked; stop N is unlocked once
  `ADVENTURE_STOPS[N-1].id` is in `game.adv.cleared`. Purely sequential, no branching
  paths.
- **`renderAdventureMap()`** — draws the stop list from `ADVENTURE_STOPS` +
  `game.adv.cleared`, a "circuit complete" banner once every stop is cleared, and
  calls `renderFence()` for the upgrade shop below the stop list.
- **Palettes**: each stop's `palette` object holds 7 hex values (`felt1`, `felt2`,
  `feltDeep`, `brass`, `brassBright`, `panel`, `panel2`) mapped in `ADV_PALETTE_VARS`
  to the actual CSS custom properties. `applyPalette(stop)` sets them as inline styles
  on `document.documentElement` when a casino is entered; `resetPalette()` removes
  them (falling back to the `:root` defaults) when the visit ends. This is the *entire*
  mechanism for each room having a distinct look — no per-casino CSS exists or should
  need to. If a new override ever looks disconnected from the rest of the theme (like
  the card panels did before `--panel`/`--panel-2` were added to the override set),
  the fix is almost always "find the CSS custom property that's still leaking through
  and add it here," not new component-specific CSS.
- **The Fence / Take / upgrades**: `game.adv.take` is a persistent currency, credited
  with net profit (`bankroll at the end minus the stake actually paid`) whenever a
  stop is won. `ADV_UPGRADES` lists the three purchasable upgrades; `game.adv.upgrades`
  tracks which are owned (booleans, one-time purchases, no stacking). `advHeatLimitFor`/
  `advStakeFor` apply the `steadyHands`/`bankroll` bonuses wherever a stop's raw
  `heatLimit`/`stake` would otherwise be used directly — **always call these, never
  read `stop.heatLimit`/`stop.stake` directly**, or a bought upgrade silently won't
  apply. The `friend` upgrade's once-per-visit "Cool It" button
  (`#btnCoolIt`/`game.adv.coolItUsedThisVisit`) resets `game.adv.heat` to 0 on demand.
- **`showStoryBeats(beats, onDone)`** — sequences an array of beats behind a single
  reused "Continue" button (`.onclick` is reassigned each call rather than
  accumulating listeners — deliberate, don't switch this to `addEventListener`
  without also removing the old handler). Toggles between the map view and story
  view; calls `onDone()` after the last beat.
- **`enterCasino(stop)`** — plays the intro story, then reconfigures the *existing*
  Play tab state for that stop (deck count, stake as `game.bankroll`, resets shoe/
  count/heat) and switches to the Play tab programmatically. This is intentional
  reuse, not a parallel implementation: the casino visit **is** a normal Play-tab
  session, just with `game.mode==='adventure'` and different starting conditions.
- **`finishAdventureVisit(outcome)`** — the reverse: restores the ordinary
  `savedFreeBankroll`/`mode:'free'`, switches back to the Heist tab, and plays the
  win/lose/heat story before re-rendering the map.
- **Heat**: `game.adv.heat` increments by `ADV_HEAT_PER_HAND` at the end of every
  resolved hand (in `finishRound()`, guarded by `game.mode==='adventure'`), capped at
  `advHeatLimitFor(stop)`. Hitting the cap before the goal ends the visit as a "heat"
  outcome (distinct from a "lose"/bust outcome) — see the `finishRound()` block right
  after the existing Challenge-mode bust check for exactly where this hooks in.
- **Persistence**: `game.adv.cleared`, `game.adv.take`, and `game.adv.upgrades` are
  saved under the same `countroom_save_v1` localStorage key Count Room already uses,
  as a new `adv` field. A live mid-visit session (current stop, heat, whether Cool It
  was used) is *not* persisted — same philosophy as the rest of the app not trying to
  resume mid-round across a reload. On load, `game.mode` never restores to
  `'adventure'` even if that's what was last saved; it falls back to `'free'`.

## Balance tuning (already done twice, worth knowing before changing it)

The heat/goal/stake numbers for every stop were stress-tested with a scripted jsdom
simulation (bet a fixed chip size every hand, play the whole visit out, repeat ~25
times per bet size, tally win/lose/heat) before shipping — twice, in fact. The first
pass on The Rusty Nickel (`heatLimit=20`, `+2`/hand, `goal=500` on a `$200` stake) made
the level nearly unwinnable, and while retuning the rest of the circuit, The Silver
Spur specifically needed a second pass (`goal` dropped from 650 to 550, `heatLimit`
raised from 40 to 45) after its first numbers came back visibly weaker than the other
three stops at the same bet sizes. **If you add a stop or change these numbers,
re-run a similar simulation before shipping** — it's very easy to accidentally make a
level unwinnable this way, and it isn't obvious from just playing it a couple of times
by hand. Current numbers land in the rough range of a 10-30% win rate at moderate flat
bets, rising with bet size, with bust as the dominant failure mode at reasonable bets
and heat as the dominant one at very small bets (not enough hands to reach the goal in
time) — that shape, not any specific percentage, is what to preserve.

**A test-harness bug to watch for if you touch the simulation script again**: match
outcomes by checking for a substring from each *stop's own, specific* win/lose story
text (see `WIN_MARK`/`LOSE_MARK` if that script still exists locally — it's not
committed, since it's a one-off tuning tool, not a real test suite), not a shared list
of phrases across stops. A shared list silently miscategorized every non-matching
outcome as a timeout during this pass, which looked exactly like impossible odds (all
"none" results) until traced back to the test script rather than the game.

## Testing approach

Same no-real-browser situation as Count Room (see that repo's notes on why). For this
layer specifically, the useful jsdom pattern is scripting a full visit end-to-end:
click into the Heist tab → click a stop's Enter button → click through
`btnAdvContinue` for the intro beats → loop betting/dealing/acting/next-round until
`advStoryView` becomes visible again (the outcome story) → match `advStoryText`
against that stop's own win/lose text to identify which outcome fired. Budget at
least ~4 loop iterations per hand and enough iterations to cover the stop's full heat
limit (e.g. 500+ for a `heatLimit` in the 40-50 range) — too small a budget produces
the exact "everything times out" false alarm described above.

## Open questions for the next pass

- No ending screen/credits beyond The Vault's epilogue dialogue — could be a distinct
  full-screen moment rather than just more story beats, if that's ever worth the effort.
- The icon/manifest are identical to Count Room's (same spade mark, same colors) — not
  yet given a distinct identity for this spinoff, even though the in-game casinos now
  each have one.
- No re-splitting/surrender in casino visits, same as Count Room's Play tab (shared
  limitation, not specific to this layer).
