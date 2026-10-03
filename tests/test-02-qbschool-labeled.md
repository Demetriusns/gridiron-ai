# Gridiron AI — vision test #2 (2026-10-02)

Source: "Aaron Rodgers vs the Browns Week 4, 2026 Analysis" — The QB
School (JT O'Sullivan, 11-year NFL QB)
(https://www.youtube.com/watch?v=x8MY9kK2AUc, 48:20). 3 pre-snap All-22
frames: ~1:43, ~15:06, ~34:59. Blind reads locked before transcript
review. O'Sullivan names coverages throughout → true labeled scoring
pending transcript.

## Blind reads

1. ~1:43 (goal-to-go): Steelers shotgun 3x1 / Cover 1 man / medium-low.
   (Noted Cover 0 also live.)
2. ~15:06 (ball ~15): Steelers 3x1, X alone to field / Cover 3 (1-high) /
   medium-low. Probabilistic framing applied (lesson from test #1).
3. ~34:59 (red zone, ball ~12): Steelers shotgun 2x2 doubles / Cover 1
   45%, Cover 3 30%, Quarters 15%, Cover 0 10% / medium-low. Press
   alignment on boundary WR drove the Cover 1 lean.

## Ground truth from Demetrius (2026-10-02)

Play 1: **starts doubles (2x2), then motions to 3x1** — machine caught
the post-motion look but missed the starting formation. Coverage call
(Cover 1 man) confirmed correct by his eye.

Lesson: pre-snap is not a still photo — motion changes the picture.
The analyzer must track formation *through* motion (pre-motion set +
post-motion set), not just read the freeze frame. Jet/fly motion to
3x1 from doubles is also a tendency tell in itself.

Play 2: **shotgun (not under center), 3x1 confirmed; RB aligned weak
(away from formation strength)** — machine got the 3x1 and the Cover 3
call right, but missed the QB depth (said under center) and never
located the RB. Coverage call confirmed correct by his eye.

Lesson: RB alignment relative to formation strength is a first-class
read (weak-side RB = pass/protection tell). The analyzer's formation
model must include backfield location, not just receiver distribution.

## Pending

- O'Sullivan transcript scorecard (browser task running).
- His grades on plays 2–3.

## O'Sullivan scorecard (transcript, 2026-10-02)

- **Play 1 (~1:43, jet sweep TD):** O'Sullivan: "**man coverage**" (1:08).
  Machine: Cover 1 man. **HIT.** (Also confirmed by Demetrius.)
- **Play 2 (~15:06, protection/sack):** O'Sullivan names no coverage in
  this window (segment is about protection busts). Unscored vs.
  O'Sullivan — but **Demetrius confirmed the Cover 3 call correct.**
  **HIT** via his ground truth.
- **Play 3 (~34:59):** O'Sullivan: "**middlefield closed. press man**"
  (34:54–35:07). Machine's lean: Cover 1 45%. **HIT on the lean** —
  which validates the probabilistic pre-snap format: the top
  probability was the right answer.

**Coverage score: 3 for 3** (two vs. O'Sullivan, one via Demetrius).

## Product gold from O'Sullivan's own mouth

- "watch the motion from the back and watch the defense not move. So,
  that's **man indicator number one**" (22:37) — encodable rule: motion
  with no defensive adjustment → man.
- "you really have to **see it post snap**... these guys will run out
  of the roof and pattern match" (29:44) — an 11-year NFL QB validating
  Demetrius's pre/post directive verbatim.
- "it presents like middlefield closed man, but **it's really zero**"
  (38:40) — disguise detection is a first-class requirement: pre-snap
  presentation vs. post-snap reality must be tracked separately.
- Tendency example: "the Browns this whole game were kind of giving
  **two middlefield closed looks**" (34:54) — that sentence IS the
  product: game-long shell tendency, one line.
