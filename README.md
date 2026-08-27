# Count Room: Heist

A lighthearted card-counting caper built on top of [The Count Room](https://github.com/SilentDark312/count-room)'s blackjack engine — work a circuit of casinos with a crew, playing real hands under each room's real house rules, trying to walk out with the goal before the pit boss gets wise to you.

**Play it live:** https://silentdark312.github.io/count-room-heist/

This is a separate project from The Count Room, not a replacement for it — the original stays exactly as it was, a straightforward blackjack + card-counting teaching tool. This one reuses that engine (rules-aware strategy, real deck-count/house-rule pairings, the whole betting/dealing/resolution loop) underneath a story layer: a map of casino stops, dialogue beats, a stake and a profit goal per stop, and a "heat" meter that ends your visit early if you push your luck too long.

*Making changes to this project? Read [`DEVELOPMENT.md`](DEVELOPMENT.md) first — it only documents what's different from the base engine. For the blackjack/strategy/counting engine itself, see [Count Room's DEVELOPMENT.md](https://github.com/SilentDark312/count-room/blob/master/DEVELOPMENT.md).*

## Status: engine proof, one stop

This is deliberately a skeleton right now: one real casino ("The Rusty Nickel") plus a locked "coming soon" placeholder, just enough to prove the loop works end-to-end — map → story → casino → win/lose story → map, progress saved locally. The full circuit (a real multi-stop arc with an ending) is a planned next pass, not yet written.

## What's inside

- **Heist tab** — the map of casino stops. Enter an unlocked one, read Frankie's intro, then you're playing real blackjack at that casino's stake, deck count, and house rules (identical rules-aware engine as Count Room's Play tab). Reach the goal and you clear the stop; bust, or let the heat meter max out, and you're sent packing (no permanent penalty — the table's still there tomorrow).
- **Learn / Strategy / Play / Trainer** — unchanged from Count Room; still available so you can study strategy or drill counting between jobs.

## Regenerating the showcase / icons

Same CI pipelines as Count Room, copied as-is — see that repo's `DEVELOPMENT.md` for how `showcase.yml`, `verify-screens.yml`, and `generate-icons.yml` work. They're unmodified here.
