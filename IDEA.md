# Gridiron AI — football film analyzer (idea, 2026-10-02)

His words: "An AI that can analyze film for American football and break
down: tendencies offensive and defensive, key players, type of offense,
type of defense, plays to review, what could work against. Almost like
how Hudl does now but more advanced."

## What it does

Upload game film → get an opponent scouting report:
- **Tendencies:** run/pass splits by down, distance, field zone, formation;
  blitz/stunt tendencies; coverage shells by situation.
- **Key players:** who makes the plays — top tacklers, pass rushers,
  playmakers; jersey numbers tied to impact.
- **Scheme ID:** type of offense (spread, air raid, power, option…), type
  of defense (4-3, 3-4, 4-2-5, coverage base).
- **Plays to review:** auto-clipped key plays with plain-English notes.
- **What could work (FINAL phase, not now):** counter-suggestions — concepts
  that attack the tendencies found. This earns its place only after play
  recognition is proven; see the phased roadmap below.

## Why it can beat Hudl

Hudl is a film library with manual tagging — teams do the work. The wedge:
- Natural-language Q&A over the film ("what do they run on 3rd and long
  past midfield?").
- Automated opponent scouting reports, not just tagged clips.
- Predictive tendency modeling (what's *likely* coming, by situation).

## Honest difficulty ladder

1. **v1 — tag-assisted analysis (buildable now):** user tags plays
   (down, distance, formation, result) or imports Hudl-style CSV; the AI
   does the math + the report + the Q&A. No computer vision needed.
2. **v2 — semi-auto:** CV suggests formation/play type, human confirms.
3. **v3 — full auto:** 22-player tracking, jersey-number OCR, formation
   recognition. Genuinely hard; research-grade.

He has his own game film (Mallorca Voltors, Bendigo Dragons) — perfect
v1 test data, and he knows what a good breakdown looks like from the
player side.

## Open questions (his call)

1. Who's it for first — him (self-scout), coaches, or a sellable product?
   ANSWERED 2026-10-02: v1 him, v2 coaches, v3 sell.
2. Does he have full game film (not just highlights) to build on?
3. How much manual tagging is acceptable in v1?

## Refined architecture (his call, 2026-10-02)

A full-fledged coach + analyst in one. It WATCHES film — Hudl film
(exported video, uploaded) — and understands what it's looking at.

- **Eyes (video understanding):** sample frames per play → vision model
  describes formation, motion, play type, result. No classic CV pipeline
  needed to start; multimodal models watch frames directly.
- **Brain (formation knowledge):** a knowledge base of formations, plays,
  and how they're run — built from coaching resources online (formation
  library: Trips/Doubles/I/Pistol/Empty; coverages 1/2/3/4/Quarters;
  fronts; route concepts). Retrieval: match what the eyes see against
  the library to name the formation and scheme.
- **Analyst:** tendencies (down/distance/zone/formation), key players by
  impact, scheme ID, auto-clipped plays to review.
- **Coach (FINAL phase):** "what could work" counters and game-plan
  suggestions, plain-English Q&A over the film. Built last — see roadmap.

## Phased roadmap (his call, 2026-10-03)

**Phase 1 — Play recognition. THE FOCUS NOW.** Name what's happening,
correctly, every snap:
- Step 0: game-state ingestion (down, distance, yard line/zone, clock,
  score, play type) — every tendency table hangs off it. Special-teams
  snaps (punts, FG/XP, kickoffs, returns) are routed OUT at the gate;
  Phase 1 covers offense/defense snaps only.
- Pre-snap: personnel ID (roster-backed; visual count wins the package
  call, conflicts tagged `UNVERIFIED_PERSONNEL`), formation including
  the backfield, motion tracked through the snap, defensive front and
  personnel answer, coverage shell as probabilities.
- Post-snap: safety rotation, linebacker drops, pass rush, route
  distribution, run fits, result — confirming or rejecting the pre-snap
  read. The confirmation step is what separates a guess from an
  analyst's call.
- Scoring the system: pre-snap coverage distributions are graded per
  snap with top-1 accuracy + Brier score against the confirmed call;
  the Head Analyst reports both across the game.
- Every player identified (jersey → roster → position); every
  out-of-position alignment flagged and labeled, with the
  routine/deviation verdict traveling as one unit with the label.
- Output: the Analyst — tendencies (down/distance/zone/formation),
  key players by name, scheme ID, auto-clipped plays to review.

Design fixes applied 2026-10-03 (council bug review): G1 scoring rule
(top-1 + Brier), G2 game-state owner (new Game-State Tracker
specialist, Step 0), G3 conflict rule (roster = identity truth,
visual = alignment truth, `UNVERIFIED_PERSONNEL` tag), G4
special-teams scope gate, G5 flag-unit handoff (label + verdict,
never bare).

**Phase 2 — Design detail. IMPLEMENTED 2026-10-03.** Four new specialists,
wired into the analysis order as steps 7–10 (post-snap, after Play Result
Confirmor; Player Identifier still runs alongside). Order is a dependency
chain — each step's inputs are settled before it runs:

- **PA/RPO Tagger** (step 7, `agents/playaction-rpo-tagger.md`) — one
  intent tag per snap (True Dropback / Play-Action / RPO / Designed QB Run
  / Draw / Screen) + fake detail (who bit) or read detail (read key,
  decision, right/wrong). Runs FIRST in Phase 2 because Protection
  Analyst needs its tag. **Resolves council G8**: intent classification
  (PA vs straight pass, QB run vs designed run) is owned here, in Phase 2.
- **Blitz Identifier** (step 8, `agents/blitz-identifier.md`) — names the
  pressure package: who blitzed (by name), from where, what kind. Fixed
  menu (exact strings, no slashes): Standard (No Blitz) / Overload /
  Zone Blitz / Creeper (Simulated Pressure) / Green-Dog / Mugged-Up A-Gap
  / Edge Blitz / All-Out (6+ Rush), with stunt games (T/E, E/T, Double
  Twist, Loop) as a fixed modifier menu. Resolves every pre-snap pressure
  indicator from Front Identifier's presentation note as SENT or DROPPED
  (unlisted rushers → UNACCOUNTED); the show-vs-send record becomes a
  disguise-rate tendency. "Standard" is base-front-relative (4 from 4-down,
  3+1 from 3-down), not a literal 4-man count.
- **Route Reader** (step 9, `agents/route-reader.md`) — names the pass
  concept from Confirmor's route shape. Fixed 18-concept menu (Mesh,
  Shallow Cross, Smash, Four Verticals, Dagger, Y-Cross, Yankee, Stick,
  Flood, Slant-Flat, Levels, Sail, Spot, China, All-Curl, Screens, Go,
  Unclassified) + motion tag (`Mesh+Jet`) + which route won vs the
  confirmed coverage and why, structurally. Takes Blitz Identifier's
  pressure call as input — a concept vs a 4-man rush grades differently
  than vs an all-out blitz.
- **Protection Analyst** (step 10, `agents/protection-analyst.md`) —
  closes the pipeline. Run-blocking scheme menu (Inside/Outside Zone, Duo,
  Power, Counter, Trap, Draw, Sweep/Pin-Pull) and protection menu (5-man
  full/half slide, 6-man, 7-man max, PA protection, Empty, Screen).
  Grades blitz pickup, diagnoses free runners (beaten man vs missed call),
  attributes every pressure/sack by name. Consumes the intent tag (step 7)
  and the pressure call (step 8) — no forward references.

Head Analyst extended: steps 7–10 in the order, four new tendency tables
(blitz rates by situation, concept frequencies vs coverage, protection +
pressure-attribution, PA/RPO rates + read keys), Phase 2 cross-checks
(blitz-vs-front, concept-vs-coverage, protection-vs-blitz, tag routing),
and design notes in Plays to Review. The Phase 3 firewall (no counters)
is unchanged — design detail sharpens the tendencies; it doesn't earn
answers yet.

**Phase 3 — "What would work against." LAST.** The Coach:
counter-suggestions and game-plan answers built on tendencies the
first two phases proved. Nobody trusts counters from a system that
can't name the coverage — that's why it's last.

## His directive (2026-10-02): the full play, pre and post

From the film tests: the analyzer reads **all 11, pre-snap and
post-snap** — never a frozen frame of just the receivers.

- **Pre-snap:** formation INCLUDING the backfield (QB depth — shotgun vs
  under center; RB location relative to formation strength — weak-side
  back is a pass/protection tell). **Personnel ID first:** count the
  backs, receivers, tight ends — 10/11/12/21/22 personnel. Personnel is
  the primary tendency key (run/pass splits, formation rates, and
  defensive answers all hang off it). Track formation *through* motion:
  pre-motion set + post-motion set (doubles → 3x1 jet motion is itself a
  tendency tell). Coverage as *probabilities* from the shell, never a
  lock (2-high could be Cover 2, 4, 6, or a rotation).
- **Post-snap:** safety movement/rotation (confirms the coverage),
  linebacker drop depth, pass rush, route distribution, run fits, result.
- The post-snap read resolves the pre-snap probabilities — that
  confirmation step is what separates a guess from an analyst's call.
- **Defensive personnel + front, same treatment:** ID the front (4-3,
  3-4, 4-2-5/nickel, 3-3-5, dime, goal-line) and the defensive answer
  to offensive personnel (11 → nickel? 12 → base?). Box count and
  shell probabilities complete the defensive pre-snap read. The
  offense-defense personnel matchup is itself a tendency key.

Test-log lessons feeding this: test-01 (don't trust raw depth for the
shell; count detached eligibles), test-02 (track through motion; locate
the RB).

Hudl note: Hudl film lives behind login with no public API — the path is
export/download the video from Hudl, upload here. Never automate against
Hudl's site without checking their terms.

## Roster-based player ID (his call, 2026-10-03)

Jersey numbers alone are just numbers — rosters turn them into players.
Per film/game, the user supplies (or the tool pulls) the team roster:
name, jersey number, listed position.

- **Number → player → position:** jersey-number OCR on frames looks up
  the roster, so the analyzer works with "M. Torres, #8, slot WR" instead
  of "the #8 guy." Personnel ID stops being pure body-counting — sum the
  listed positions to confirm 11/12/21 groupings.
- **Flag and label every out-of-position alignment:** whenever a player's
  alignment doesn't match his listed position, it gets flagged and
  explicitly labeled — "backfield WR," "RB split wide," "inline TE split
  out," "DL standing up on the edge," etc. Nothing unusual slides by
  unlabeled. The tendency engine then sorts them: routine for this
  player/team (Deebo's backfield package — logged as tendency) vs.
  genuine deviation from his established usage (first time all game —
  surfaced to the analyst). The label is always there; the urgency
  depends on the player's norm.
- **Defensive side:** ID the blitzers and coverage players by name —
  "#5 is their nickel; he blitzed 4 of 6 third downs" beats "the nickel
  blitzed."
- **Key players get names:** the "who makes the plays" section ties
  impact stats to actual players, which is what a scouting report needs.
- **Caveat:** rosters go stale (transfers, number changes, week-to-week
  depth moves). Use the game-date roster; when a number isn't on it,
  fall back to visual personnel ID and mark the player unidentified.

## Honest hard parts

- Play segmentation (where one play ends and the next begins) is the
  unsexy hard problem; v1 may need manual play markers.
- Video is heavy: storage + per-frame vision cost. Budget per game early.
- Formation recognition accuracy is the make-or-break; the knowledge base
  quality decides it.

## Standing note

Separate project from Orion. Business side stays personal, same rule.
