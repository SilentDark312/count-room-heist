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

- **`ADVENTURE_STOPS`** — the only place "content" lives. Each stop is
  `{id, name, blurb, numDecks, stake, goal, introStory, winStory, loseStory}`, where
  `introStory`/`winStory`/`loseStory` are arrays of `{who, text}` beats. A `locked:true`
  stop (like the placeholder second entry) renders greyed-out on the map with no
  Enter button. **Adding a new stop is just adding an entry to this array** — no other
  code should need to change for ordinary content additions.
- **`renderAdventureMap()`** — draws the stop list from `ADVENTURE_STOPS` +
  `game.adv.cleared`.
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
  `ADV_HEAT_LIMIT`. Hitting the cap before the goal ends the visit as a "heat" outcome
  (distinct from a "lose"/bust outcome) — see the `finishRound()` block right after
  the existing Challenge-mode bust check for exactly where this hooks in.
- **Persistence**: only `game.adv.cleared` (which stops are cleared) is saved, under
  the same `countroom_save_v1` localStorage key Count Room already uses, as a new
  `adv` field. A live mid-visit session (current stop, heat) is *not* persisted —
  same philosophy as the rest of the app not trying to resume mid-round across a
  reload. On load, `game.mode` never restores to `'adventure'` even if that's what
  was last saved; it falls back to `'free'`.

## Balance tuning (already done once, worth knowing before changing it)

The heat numbers aren't arbitrary guesses — they were stress-tested with a scripted
jsdom simulation (bet a fixed chip size every hand, play the whole visit out, repeat
40 times, tally win/lose/heat) before shipping, because the first pass
(`ADV_HEAT_LIMIT=20`, `ADV_HEAT_PER_HAND=2`, `goal=500`) turned out to make the level
nearly unwinnable — heat capped almost every run at moderate bet sizes before the
bankroll math even got a chance to play out. Current numbers
(`ADV_HEAT_LIMIT=40`, `ADV_HEAT_PER_HAND=1`, `goal=400` on a `$200` stake) land
around a 10% win rate at conservative flat $25 bets and ~35-40% at $100 flat bets, with
bust (not heat) as the dominant failure mode at reasonable bet sizes — a real, bet-size-
sensitive difficulty rather than a hard wall. **If you add a new stop or change these
numbers, re-run a similar simulation before shipping** rather than eyeballing it; it's
very easy to accidentally make a level unwinnable this way, and it isn't obvious from
just playing it a couple of times by hand.

## Testing approach

Same no-real-browser situation as Count Room (see that repo's notes on why). For this
layer specifically, the useful jsdom pattern is scripting a full visit end-to-end:
click into the Heist tab → click a stop's Enter button → click through
`btnAdvContinue` for the intro beats → loop betting/dealing/acting/next-round until
`advStoryView` becomes visible again (the outcome story) → read `advStoryText` to
identify which outcome fired. That loop is also how the balance simulation above was
built — worth reusing rather than re-deriving if you touch this again.

## Open questions for the next content pass

- The full circuit (a real multi-stop arc, 3-4 casinos, an actual ending) hasn't been
  written yet — this phase was deliberately scoped to "prove the engine, one
  placeholder casino" before investing in a full story.
- No visual distinction between casinos yet (e.g. a palette swap per stop) — cheap to
  add later, skipped for the engine-proof phase.
- The icon/manifest are identical to Count Room's (same spade mark, same colors) —
  not yet given a distinct identity for this spinoff.
