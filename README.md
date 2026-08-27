# Count Room: Heist

A lighthearted card-counting caper built on top of [The Count Room](https://github.com/SilentDark312/count-room)'s blackjack engine — run a circuit of four casinos with a crew, playing real hands under each room's real house rules, trying to walk out with the goal before the pit boss gets wise to you.

**Play it live:** https://silentdark312.github.io/count-room-heist/

This is a separate project from The Count Room, not a replacement for it — the original stays exactly as it was, a straightforward blackjack + card-counting teaching tool. This one reuses that engine (rules-aware strategy, real deck-count/house-rule pairings, the whole betting/dealing/resolution loop) underneath a story layer: a map of casino stops, dialogue beats, a stake and a profit goal per stop, a "heat" meter that ends your visit early if you push your luck too long, and a per-casino visual identity.

*Making changes to this project? Read [`DEVELOPMENT.md`](DEVELOPMENT.md) first — it only documents what's different from the base engine. For the blackjack/strategy/counting engine itself, see [Count Room's DEVELOPMENT.md](https://github.com/SilentDark312/count-room/blob/master/DEVELOPMENT.md).*

## What's inside

- **The circuit** — four casinos, unlocked in order: **The Rusty Nickel** (2 decks, small-time), **The Silver Spur Saloon** (4 decks), **The Riviera Room** (1 deck, high-roller), and **The Vault** (8 decks, the campaign finale with an epilogue). Each has its own stake, profit goal, house rules (the same rules-aware engine as Count Room's Play tab), and a distinct felt/brass color identity.
- **Heat** — a per-visit pressure meter. Reach the goal before it maxes out or you're sent packing — no permanent penalty, the table's still there tomorrow.
- **The Fence** — net winnings from a cleared casino become permanent "Take" you can spend between jobs on upgrades: more heat capacity, a bigger starting stake, or a once-per-job "cool it" that resets heat to zero on demand.
- **Learn / Strategy / Play / Trainer** — unchanged from Count Room; still available so you can study strategy or drill counting between jobs.

## Regenerating the showcase / icons

Same CI pipelines as Count Room, copied as-is — see that repo's `DEVELOPMENT.md` for how `showcase.yml`, `verify-screens.yml`, and `generate-icons.yml` work. They're unmodified here.
