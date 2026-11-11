# Loopback spec (v1)

Status: approved by the owner on 2026-11-10: he named it Loopback and took every other default.
The owner created the repo the same day. Building waits for its turn in the queue. Red-teamed once
with Fable (fixes listed at the end).

## What it is

A snake-style game for `loopback.prignano.ai`. You steer a line that moves on its own
around a grid. Eating a dot makes you one segment longer. Running into a wall or yourself ends
the game.

**The twist: you catch things by drawing loops with your body.** Sparks drift around the board
and slip away when your head gets close, and touching one head-on only makes it jump away. The
only way to catch a spark is to curl your body around it until it's fully surrounded. Two sparks
in one loop score four times as much as one. Being long makes big loops possible, and also makes it easy to
trap yourself, so every game is a trade between growing and staying safe.

The name is a programming pun, in the family of Infinite Recursion and Zero Sum: "loopback" is
the network address that sends a message straight back to the machine that sent it, and here your
own body loops back on itself to catch what's inside.

It's plain JavaScript and canvas with no build step, served as static files. It installs as an
app (PWA) on iPhone, iPad and Android from day one. Made for all ages: kids will play it.
Nothing leaves the device.

## Decisions so far

| # | Question | Answer |
| --- | --- | --- |
| 1 | Kind of game | Snake-style, with a twist (owner, 2026-11-09) |
| 2 | Twist | Loop capture, A below (owner took the default, 2026-11-10) |
| 3 | Name | **Loopback** (owner, 2026-11-10) |
| 4 | Look | Original. No product names from the genre's history in the game, no copied art or sounds (see Prior art) |
| 5 | Classic no-sparks mode | Not in v1 (owner took the default, 2026-11-10) |
| 6 | Phone controls | Arrow pad shown by default on touch screens, swipes also work, a setting hides the pad (owner took the default, 2026-11-10) |
| 7 | Home | Repo `nickprignano/loopback`, served at `loopback.prignano.ai`, deployed by Workers Builds on each merge to `main`. The owner creates the repo and adds the domain. Launch ends with the `content/games.json` PR in `prignanodotai-web` |
| - | License | MIT. Every part is our own work |

## Twists considered

| Twist | How it plays | Verdict |
| --- | --- | --- |
| **A. Loop capture (chosen)** | Fleeing sparks can only be caught by circling them with your body. Multiple sparks in one loop multiply the score | Changes how you move, not only what you dodge. Easy to explain in one sentence, fun to master, and the snake's length becomes a tool instead of only a hazard. Proven fun in small games (see Prior art), and ours adds the fleeing and herding |
| B. Bent worlds | Each level joins the edges differently: wrap around, then a Möbius strip (leaving the right edge brings you in on the left, upside down), then stranger shapes | Mind-bending and a little educational, but the core play stays classic snake, and younger kids may just feel lost |
| C. Past selves | Each 20-second round, your path from the last round replays as a ghost snake you must avoid | Strong idea, but the board crowds fast and it's hard to read on a phone. It also overlaps the "recursion" theme Infinite Recursion already uses |
| D. Color chains | Segments take the color of what you ate, and three of a color in a row pop off | Rejected. New segments always join at the same end, so the best play is "always eat the same color", which turns the game into nothing. Removing that choice makes it pure luck |

The rest of this spec assumes A.

## Prior art and licenses

Checked 2026-11-10 (our KB prior-art script, GitHub search, web search).

| Source | License | What we take |
| --- | --- | --- |
| The genre | Not protectable | Moving line, grows when it eats, dies on walls and itself. Began in 1976 arcades and became famous on phones in the late 1990s. Mechanics are free to use; names, art and sounds are not |
| [Blasnake](https://abagames.itch.io/blasnake) (abagames; source in `abagames/vibe-game-coding-testbed`) | MIT (checked 2026-11-10) | The idea that circling things with your body captures them, with a bonus for many at once. Idea only, no code; credited on the About screen. Ours differs: nothing explodes, the targets flee, walls don't count as part of a loop |
| Libble Rabble (1983 arcade) and the Qix family | all rights reserved | The older idea of enclosing moving things with your line. Ideas only; their names go on the banned-words list |
| Many game-jam snake games with loop or "ouroboros" ideas (a 2025 jam theme was "Loop") | mixed | Nothing. Confirms looping is a genre idea, not one game's |
| [patorjk/JavaScript-Snake](https://github.com/patorjk/JavaScript-Snake) and other MIT snake games on GitHub | MIT | Nothing. The core is short to write ourselves |
| Zero Sum and Infinite Recursion (ours) | MIT | Flash limiter, banned-words test, seeded pure game module tested in Node, Reduce effects switch, control and pause rules |
| Pinball PR #7 (ours) | ours | The PWA recipe (also in ai-brain's prompt library) |

**Name check:** "Closure" (the programming term for a function that captures what's around it)
was the first idea, but a well-known 2012 puzzle game already uses it. "Loopback" is the owner's
pick (2026-11-10). A 2022 PC arcade shoot-'em-up on Steam is also called Loopback (with a sequel),
and a few small itch.io games use the word. None is a snake-style game, and the name is a common
networking term, so we accept the overlap, as Infinite Recursion did. The About screen never
suggests a link to any of them.

## How a game plays

All timings are counted in **moves**, not seconds, so a seeded game is the same at any speed and
pausing changes nothing.

### Board and movement

- A **20 × 20** grid of playable cells, the same on every screen size so scores compare fairly.
  The wall sits outside those 400 cells.
- You start 4 segments long in the middle of the left side, moving right.
- **Speed:** **Normal** starts at 7 moves a second and gets 0.5 faster every 5 dots, up to 12.
  **Relaxed** stays at 5 a second, for younger kids. Each has its own score table.
- Turns are queued: up to 2 turns pressed within one move both happen, in order, so quick
  "up then left" works. A turn straight back into yourself is ignored.
- Moving into the cell your tail is leaving on that same move is safe, as in most snake games.

### Dots

- There is always one dot on the board. Eating it adds a segment and 1 point.

### Sparks

- The first spark appears once you are **10 segments long**. After that a new one appears every
  **60 moves** (a steady counter that ignores catches) while there are fewer than **3** on the
  board.
- A spark is a small four-pointed star that turns slowly. It lasts **180 moves**. In its last 20
  it dims step by step (no blinking), so you can see it's about to go. A dimming spark can still
  be caught.
- Sparks live only in the **inner 18 × 18**: they never appear in, or move into, a cell touching
  the wall, so fleeing can't park one where a loop can't reach.
- **Fleeing:** a spark is "alarmed" when your head is within 3 cells (counting diagonals), and
  shows a thin ring around itself while it is. An alarmed spark moves on every move whose number
  is a multiple of 3 (one game-wide counter), one cell up, down, left or right (never diagonally),
  to the allowed cell farthest from your head by straight-line distance; ties go to the seeded
  random number generator. A calm spark drifts one random allowed cell every 6th move. Allowed
  cells are empty, inside the inner 18 × 18, not a dot, and still connected to the outside. A
  spark with no allowed cell stays put.
- **Head-on:** if your head enters a spark's cell, the spark jumps to a random open cell at least
  5 away. No points. Only loops score.
- **Catching** uses one rule: a "4-connected fill". Starting from the area outside the wall, the
  game spreads through empty cells up, down, left and right, never through your body. Any spark
  or dot in a cell the fill can't reach is caught.
  - Two body cells that touch only at a corner still block the fill, so the smallest loop is 7
    segments around one cell.
  - **Only your body counts.** The area outside the wall is where the fill starts, so a spark
    pinned between you and the wall is not caught. (Sparks can't reach the edge cells anyway.)
  - Score: **10 × (number of sparks caught on that move)²**, so 1 spark is 10, 2 are 40, 3 are
    90. Each caught spark also adds a segment, and each counts as a dot toward the next speed
    step, so catching makes the game harder the same way eating does. Each dot caught in a loop
    is eaten as usual (a segment and a point each).

### The order of a move

1. Apply the next queued turn.
2. Move the head. If it hits the wall or the body (except the cell the tail is leaving), the game
   ends.
3. If the head is on the dot, grow and score it. If it is on a spark, the spark jumps away.
4. Move alarmed and drifting sparks whose turn it is (they see the head's new position).
5. Run the fill. Score and remove caught sparks and dots; add their segments to the tail over the
   next moves.
6. Count down spark lives and spawn what's due.

**Spawning** puts a new dot or spark in a random open cell (empty and reachable by the fill) at
least 3 from the head, and for sparks inside the inner 18 × 18. If no cell is 3 away, it uses any
open cell; if there is none, it skips this move and tries again next move.

### Ending

The game ends when your head hits the wall or your body. The score, best score and a **Play
again** button appear.

### Why these numbers

A quick simulation (2026-11-10, a simple bot that circles the nearest spark) caught a fleeing
spark in about 20 to 30 moves when the snake was 12 to 16 segments long and the spark moved every
3rd move. At 8 segments the bot mostly failed, which is why sparks wait until 10. When the spark
moved every 2nd move, the bot's catches dropped sharply. The Fable red team then showed that a
bot just chasing head-on caught every spark in a median of 14 moves, which is why head-on now
scores nothing. The build tunes the numbers with the balance test below, not by feel.

## How it looks and feels

### Art

- Line art in prignano-theme colors, following light and dark mode. The snake is one continuous
  rounded stroke in the theme accent, with a slightly wider head and two small eyes. Dots are small
  filled circles; sparks are outlined four-pointed stars, so shape tells them apart, not only
  color.
- **A catch:** the closed-off area fills with a soft tint, never more than 25% opacity, that
  rises over 150 ms and fades over 600 ms, and caught sparks spiral to the center of the loop and
  vanish. Nothing explodes.
- The icon is an original drawing: a curled line around a small star.

### Flash safety (from Zero Sum)

No full-screen flashes and nothing saturated red. The pure flash limiter allows at most 3 glow
events a second, and every loop tint counts as one; a tint over the limit is skipped. A tint may
be large, but it stays at 25% opacity or less and fades slowly rather than flashing. **Reduce effects** (a switch on the title screen) or `prefers-reduced-motion`, watched
live, turns off the tint, the spiral and the spark's turning.

### Controls

- **Phone:** on a 360 px phone a cell is about 18 px, so a finger on the board would hide the
  loop you're drawing. Two ways to turn, both always on:
  - A **four-arrow pad** under the board (phones have room below a square board in portrait),
    each button at least 56 px. Shown by default on touch screens; a setting hides it.
  - **Swipes anywhere on the screen**, including below the board. A turn fires during the swipe
    as soon as the finger has moved 20 px, and a new direction can fire without lifting, so
    quick zigzags work. (A 7-segment loop needs a turn every 2 moves, about 285 ms at the
    starting speed and 165 ms at the top.)
  - `touch-action: none`, no double-tap zoom or pull-to-refresh. Pause is a 44 px button outside
    the board.
- **Keyboard:** arrows or WASD to turn, P or Escape to pause, M to mute.
- The game pauses when the tab is hidden. Resuming, from that or from a manual pause, starts a
  3-second countdown.
- The first game shows one animated "How to play" card: eat dots to grow, circle a spark to catch
  it, and **walls don't count**, only your body. It's in the title menu afterward.

### Accessibility

Menus, how to play, scores and the game-over screen are real DOM text. A polite live region
announces the score at game over and each catch ("Caught 2 sparks, 40 points"). The game itself is
fast and visual; Relaxed and the arrow pad are the ways in for players who need them.

### Sound

Synthesized with the Web Audio API, so there is nothing to license: a soft blip for a dot, a
short rising run for a catch (one note higher for each spark in the loop), a low soft tone at game
over. Muted until the first tap; the mute setting is saved on the device.

## Scores and privacy (kids will play)

- A top 10 for each speed on the device: score and date. No names or initials.
- Nothing leaves the device: no accounts, analytics, ads, server or network calls beyond the
  pinned theme files. The prignano-theme header loads from jsDelivr pinned to a commit SHA, as the
  other games do, and is cached for offline use.
- Settings and scores stay in `localStorage`. "Delete my data" in settings clears them.
- A short About screen says this in plain words, with the MIT notice, a link to the source, and
  the credit to Blasnake for the loop-capture idea.

## Installable app

Pinball PR #7's recipe: manifest (`display: standalone`, no orientation lock), original icons in
any, maskable and Apple touch sizes, iOS home-screen tags, a service worker that caches every
served file with its list stamped by `tools/sw-version.mjs` and a test that fails when the list is
stale, an Update button on the title screen, safe-area padding, the site header hidden in the
installed app, and sound resumed after an iPhone locks.

## Hosting

Cloudflare Workers static assets, no Worker script (`wrangler.jsonc` named after the repo,
`.assetsignore` leaving out tests, tools and docs), Workers Builds deploying each merge to `main`.
The owner creates the repo and adds the custom domain. Claude writes `docs/DEPLOY.md` the way
Infinite Recursion's is written and changes nothing on the account without his yes. The last
step of `docs/DEPLOY.md` is the `content/games.json` PR in `prignanodotai-web`: the game goes in
hidden with an empty url when the repo is created, and the launch PR sets its url and `live`.

## Files

```
index.html, styles.css, manifest.webmanifest, sw.js, _headers, icons/
src/game.js      state and step(state, turns): movement, growth, death, scoring (pure, seeded)
src/capture.js   which cells are closed off by the body (pure)
src/sparks.js    spawning, fleeing and drifting (pure)
src/draw.js      canvas drawing, catch effect, flash limiter
src/input.js     swipe, button pad, keyboard, turn queue
src/audio.js     synthesized sounds, mute
src/scores.js    top 10 per speed on the device
src/main.js      screens: title, how to play, game, game over, scores, settings, about
wrangler.jsonc, .assetsignore, docs/DEPLOY.md, tools/sw-version.mjs
test/            node --test
```

## Tests

`node --test` covers:

1. **Movement:** turn queue of 2, reversal ignored, wall and body deaths, the tail-leaving cell is
   safe, growth on a dot, speed steps (sparks count as dots).
2. **Capture:** the 7-segment loop catches (body cells touching at a corner seal); body plus wall
   never catches; two separate pockets on one move both count; dots inside a loop are eaten; the
   score is 10 × n², and 3 sparks in one loop is reachable in a scripted game.
3. **Sparks and the move order:** sparks never enter the body, a dot, an edge cell or a
   closed-off cell, and never move diagonally; they flee and drift on the right moves; a head-on
   touch scores nothing and the spark jumps away; spawning falls back and skips as written; lives
   and spawns are counted in moves.
4. **Balance:** a reference bot that circles sparks (and avoids its own body, so a failed catch
   is the game's fault, not the bot's) plays 300 seeded games each at length 10 and 12. It catches
   a lone spark within 60 moves between 40% and 85% of the time: catchable, but not free. Over
   long seeded games, the circling bot earns more points per 100 moves than a bot that only eats
   dots.
5. **Determinism:** the same seed and the same turns always give the same game.
6. **Flash limiter:** never more than 3 glow events a second, loop tints included, and Reduce
   effects gives zero.
7. **Words:** a short banned list, stored encoded so the repo never spells it out (the genre's
   famous product names, the phone maker most linked with it, and the older enclosing games
   named in Prior art), appears in no shipped file.
8. **PWA:** the service worker's file list is current.

**Done means:** all tests pass in CI, and before each build PR the build thread runs a scripted
Playwright check (not in CI, like the other games) at phone and desktop sizes: a seeded game that
eats dots, catches a spark with a loop, and dies; at phone size, a 7-segment loop drawn with
swipes alone and another with the pad; and Relaxed. Installing and
playing offline on real phones is a manual check listed in `docs/DEPLOY.md`.

## Pull requests

After the owner creates the repo:

1. **Spec:** this file as `docs/SPEC.md`, `AGENTS.md`, `CLAUDE.md`, MIT `LICENSE`. Plus the hidden
   `games.json` entry in `prignanodotai-web`.
2. **Build 1, the game:** `game.js`, `capture.js`, `sparks.js`, plain drawing, input, scores, CI,
   tests 1 to 5. Playable but plain.
3. **Build 2, polish and launch:** art, catch effect, flash limiter, sound, how to play, About,
   the PWA recipe, `wrangler.jsonc`, `docs/DEPLOY.md`, tests 6 to 8.

Then the Cloudflare setup (with the owner's yes) and the launch `games.json` PR.

## Not in v1

The other twists (a later mode, like Divide in Infinite Recursion), a classic no-sparks mode
(owner said not in v1), levels or mazes, power-ups, two players, online play, a shared leaderboard,
music, daily challenges.

## Fable red team (2026-11-10)

Fable found 13 problems. All are fixed above:

1. Head-on catching was easy (a simple chaser caught every spark, median 14 moves), so the twist
   was optional. Head-on now scores nothing and the spark jumps away.
2. Fleeing pushed sparks to the edge, where walls-don't-count made them uncatchable. Sparks now
   stay in the inner 18 × 18.
3. "7-segment loop" and "a diagonal gap doesn't close" contradicted each other. Now one written
   rule: a 4-connected fill, corner-touching cells seal, sparks move in 4 directions only.
4. Difficulty could stall by farming sparks. Caught sparks now add a segment and count toward
   speed.
5. With 2 sparks at most, "3 are 90" was unreachable and doubles were luck. Now up to 3 sparks,
   each lasting 180 moves, with a new one every 60.
6. Timings in seconds broke the seeded, pausable game. Everything is now in moves.
7. The order within a move and the spark rules were unclear. Added "The order of a move" and the
   exact flee, tie and stuck rules.
8. Spawning had no fallback and the wall's position was unclear; the win screen was unreachable.
   Added the fallback, put the wall outside 20 × 20, cut the win screen.
9. Swipes on an 18 px grid hide the loop and are too slow for tight loops. The pad is now on by
   default for touch, and swipes fire mid-gesture anywhere on screen.
10. The balance test couldn't fail in the right way. It now has a 40 to 85% band at lengths 10
    and 12 and compares points per move against a dots-only bot.
11. Large loop tints weren't covered by the flash limiter. They are now, with a slower fade, and
    every resume gets the countdown.
12. Prior art: Blasnake turned out to be MIT (still idea only), and Libble Rabble and Qix were
    added as older enclosing games.
13. Scope: the 3-screen how-to is now one card that says walls don't count; length dropped from
    the score table; added the alarm ring and the dimming cue.
