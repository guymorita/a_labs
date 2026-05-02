# Daily Games Portfolio — Plan

Working doc. Edits welcome. Last updated 2026-05-02.

## What we're building

A portfolio of short, daily, web-based puzzle games — one new puzzle per day per
game, each playable in under five minutes. Built in the lineage of Wordle,
Worldle, Enclose Horse, MapTap, and EthnoGuessr.

The first game is a **yearbook celebrity guesser**: you see a yearbook-era photo
of a famous person, and try to identify them. Scored on **speed and accuracy**.
Five (or fewer — TBD) photos per day. Real-time scoring is the differentiator —
almost every other daily game in the genre is turn-based.

Long-term: a small portfolio of distinct games sharing one brand, one domain,
one infra. Each game stands on its own; the portfolio compounds via cross-promo
and a shared cross-game streak.

## Why this, why now

- The daily-puzzle format has a proven viral primitive (the emoji-grid share)
  and a sticky daily-reset retention mechanic.
- Most 2022-era Wordle clones are dead; the survivors went deeper, not wider.
  There's still room for high-quality entrants in unclaimed niches.
- The author cares about novelty over money. The bar is "this hasn't been done
  well before" — not "this clones a winner."
- A skilled solo engineer can ship a v1 in a day. Iteration loop is fast enough
  to test multiple game concepts cheaply.

## North-star principles

In priority order. When two conflict, the higher-ranked one wins.

1. **Novelty.** Every game must do something the genre hasn't seen, or hasn't
   seen done well. No skinned Wordles.
2. **Difficulty, biased hard.** Hard games are stickier. The hook is "I can't
   believe I missed that," not "I won." Wordle and Enclose Horse both pass this
   test; cheap trivia does not.
3. **Time-to-share.** From "land on page" to "share string on clipboard"
   should feel instant. Share artifact must be non-spoilery and tweet-shaped.
4. **Daily re-trigger without notifications.** Streaks, the share habit, and
   the reset cadence carry the user back. No push-notification prompts.
5. **Performance as a feature.** Sub-30KB JS per game. Lighthouse 100 on hub,
   95+ on every game page. Free or near-free at any traffic scale.
6. **Monetize last.** Ads only after ~10k DAU on a given game. Until then, the
   game itself is the marketing — protect the experience.

## Target audience

- **Primary:** people who already play 2–4 daily games. They have the habit
  loop installed; we just need to earn a slot in their morning rotation.
  Demographic skew: 25–55, English-speaking, smartphone-during-coffee crowd.
- **Secondary:** the "saw a tweet, played once" funnel. Largest top-of-funnel
  but lowest retention. We design for the primary and accept the secondary as
  bonus.
- **Geographic:** US-first by traffic, but the format is globally accessible.
  Yearbook game has a US/UK celebrity bias; later games may localize.

## Vibe

- Quirky, clever, a little weird. Closer to Enclose Horse than to NYT Games.
- Confident copy. Short. No marketing-speak. No exclamation points in body
  copy. The game's title and tagline do the talking.
- Visual: clean, generous whitespace, one strong accent color per game,
  monospace for scores/share strings. Looks at home on Hacker News.
- The brand is not loud. The games are loud. Domain and hub fade into the
  chrome; the puzzle is the product.
- No emoji-spam, no gamification-speak ("level up!"). Tone is wry, not
  enthusiastic.

## Inspiration

| Game | What it does well | What we steal |
|------|-------------------|---------------|
| **Wordle** | The viral primitive: emoji-square share string. Daily cadence. Single puzzle. | Share string format. Daily reset. One-shot mentality. |
| **Worldle** (Teuteuf) | Took the Wordle format into a new domain (geography). Built a real portfolio over years. | Portfolio mindset. Subdomain/hub structure. Premium tier for archive. |
| **Enclose Horse** | Genuinely novel mechanic. Quirky brand. Didn't chase money. Made HN front page. | Domain-as-joke energy. Refusing to be generic. Community level editor as content multiplier. |
| **MapTap** | 5-locations-per-day pacing. Real-time scoring (closer = better). Premium archive. | Multi-prompt-per-day structure. Speed/accuracy scoring. |
| **EthnoGuessr / hbd.gg** | Single domain, multiple games as subpaths. Provocative niche. Tight gameplay loop. | The subpath portfolio structure. The "one short brand domain" decision. |
| **Heardle** | Speed-as-mechanic in a daily game. Sold to Spotify. | Validation that real-time scoring works as a daily-game format. |

## Game #1: Yearbook celebrity guesser (working title)

**Concept:** You see one yearbook-era photo of a famous person. You have a
fixed time window to type their name. The closer you are (Levenshtein-like
fuzzy match) and the faster you guess, the higher your score.

**Open questions:**
- One photo per day, or a small set (3–5)?
- Time pressure: hard timer (e.g. 30s) or soft (decay-scoring)?
- What's the share string look like? Numerical scores share well; we want a
  format that's instantly recognizable as "ours."
- Fail state: what does losing feel like? Must still be share-worthy.
- Photo sourcing: yearbook IP is gray. We need a sustainable sourcing process
  and a takedown protocol from day one.

**Why it might work:**
- Speed-as-mechanic is rare in the daily-game genre.
- Yearbook photos have built-in "wait, that's THEM?" curiosity.
- Genuinely hard — most photos won't be obvious.
- Share string is naturally numerical and tweet-shaped.

## Structure

### URL strategy

Single short brand domain. Each game lives at `brand.tld/game-name`.

- One brand to drive traffic to.
- SEO equity compounds across games.
- One deploy pipeline, one DNS, one analytics view.
- Shared cross-game streak ("portfolio loyalty") becomes a real retention
  asset.
- Escape hatch: if any single game becomes 10× the others, buy a vanity domain
  that 301-redirects to the canonical subpath.

Domain TBD. Shortlist leans toward short/quirky `.gg` interjection-style names
in the spirit of `hbd.gg`, e.g. `hmm.gg`, `reset.gg`, `huh.gg`, `oof.gg`,
`dang.gg`. Final choice deferred until v1 of game #1 is shippable; until then
ship on a free `*.pages.dev` subdomain.

### Homepage

Designed as a **launcher**, not a landing page. Most traffic skips the homepage
(Worldle is 58% direct to game URLs); the homepage exists for:
- People who heard the brand vaguely and typed in just the root.
- Returning players choosing what to play next.
- Share-tweet bystanders who clicked the brand instead of the game.

Above-fold spec:
- Header: brand · cross-game streak · "today: N of M played"
- Tile grid of today's games, each showing per-game streak and played/unplayed
  status (read from `localStorage`).
- Mini calendar: which days you played at least one game.
- No marketing copy, no newsletter signup, no login wall.

When only one game exists, ship it at `/` directly. Move it to `/yearbook`
when game #2 launches; promote the launcher to `/`. Five-minute migration if
routing is set up correctly from day one.

### Tech stack

- **Framework:** Astro + Solid islands. Hub page ships 0KB JS; each game route
  hydrates one canvas island. Mixing per-island frameworks stays an option.
- **Language:** TypeScript, strict.
- **Hosting:** Cloudflare Pages. Unlimited bandwidth on the free tier; no
  per-game cost scaling.
- **Backend:** Cloudflare Workers (lazy — added only when needed). D1 for
  optional global leaderboards.
- **State:** `localStorage` for streaks, played-today, share strings. No
  account required for v1.
- **Daily puzzles:** pre-generated as static JSON (one file per day or one
  manifest). Client picks today's based on user-local date.
- **Analytics:** Cloudflare Web Analytics (free, ~4KB, no cookies, gives Core
  Web Vitals out of the box) plus a ~30-line custom event logger via
  `navigator.sendBeacon()` to a Worker → D1 for game-specific events
  (started/won/lost/shared).
- **Error reporting:** inline ~500-byte handler posting to a Worker endpoint.
  Lazy-load Sentry only after a first error if/when richer reporting is
  needed.

### Repo structure

Monorepo from day one, even with one game. Cost is minutes; payoff is real
when game #2 ships.

```
apps/
  _hub/        Astro app for the launcher homepage
  yearbook/    Astro app for game #1
packages/
  shared/      Share-string lib, date utils, telemetry client, UI primitives
  puzzles/     Daily-puzzle generation + pre-built JSON
infra/
  workers/     Cloudflare Worker(s) for analytics + error endpoints
```

### Performance budget

- JS per game route: < 30KB gzipped.
- Lighthouse Performance: 100 on hub, ≥ 95 on game pages.
- LCP < 1.5s on mobile 4G. INP < 200ms. CLS < 0.05.
- First share-string-copyable interaction: < 1s after game-end.

## Phases

1. **v1 (week 1):** ship the yearbook game on `*.pages.dev`. No hub, no
   analytics, no error reporting. Just the game and its share string. Send to
   ten friends. Iterate on the core loop only.
2. **v1.1 (week 2–3):** wire up Cloudflare Web Analytics + custom event
   logging. Add `localStorage` streaks. Ship to a real domain.
3. **v2 (when game #1 has a working loop):** build the hub launcher, plan
   game #2, lift shared code into `packages/shared`.
4. **Monetization (when first game crosses ~10k DAU):** post-game interstitial
   ads via a casual-game-friendly network (Snigel / Publisher Collective).
   Optional premium tier for archive + ad-free.
5. **AI growth engine (parallel, starting after v1):** automated puzzle
   generation, OG share-image generation per puzzle, SEO content for puzzle
   archives, automated difficulty A/B tests based on win-rate telemetry.

## Non-goals

- Native mobile apps. Web-first until traffic justifies the app-store
  overhead. Teuteuf only built native after Worldle was clearly dominant.
- Accounts / login. Anonymous `localStorage` is enough for v1 and v2.
  Optional account comes later for cross-device sync.
- Multiplayer / real-time / chat.
- Push notifications.
- Ads, paywalls, or upsells before product-market fit.
- Generic word/letter games. The genre is saturated and we have no edge there.

## Open questions

- Domain: pick before v1.1. Confirm availability of shortlist; budget for one
  premium acquisition if a top pick is taken.
- Yearbook photo sourcing: legal review of approach before scaling beyond
  friends-and-family test.
- Scoring formula: speed/accuracy weighting needs playtesting. Probably
  exponential decay on time, fuzzy match on name.
- Share string format: needs to be visually distinct from Wordle's grid
  while still being non-spoiler and copy-pasteable.
- AI growth engine: what's the cheapest, smallest first experiment?
