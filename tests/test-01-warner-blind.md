# Gridiron AI — vision test #1 (2026-10-02)

Source: "49ers Utilizing the Middle of the Field | Kurt Warner x QB Confidential"
(https://www.youtube.com/watch?v=w0kVwIB09qM, 5:34). 3 pre-snap All-22
frames captured at 0:00, 1:40, 4:07. Blind reads done before transcript
review.

## Blind reads (structure: formation / coverage / confidence)

1. 0:00 (red zone, ball ~22): 49ers shotgun 3x1 trips / Cover 2 (2-high) /
   medium.
2. 1:40 (midfield, ball ~35): 49ers under center 2x2 / Cover 2 or Quarters
   (2-high) / medium-low.
3. 4:07 (ball ~40): 49ers shotgun 2x2 / Cover 3 (1-high) / low-medium.

## Scorecard vs. Warner

Warner does NOT name explicit coverages in this video — it's about
middle-of-field throw difficulty, not coverage IDs. So the labeled track
did not materialize here.

- Play 3: Warner says Purdy is "reading this safety back here" (singular)
  → single-high structure. Matches the Cover 3 structural read (1-high).
  Soft hit on structure; exact coverage unverified.
- Plays 1–2: no coverage named in narration → remain unscored (blind only).

## Ground truth from Demetrius (2026-10-02, he played DB)

Play 1 correction — his read: **shotgun 2x2 (doubles), man coverage with
one high safety (man-free / Cover 1)**. Machine had said: shotgun 3x1
trips, Cover 2.

Machine error analysis: the 2-high read was fooled by a defender at depth
who wasn't a true half-field safety — depth alone doesn't make a shell.
And the trips call miscounted detached receivers (2x2 doubles, not 3x1).
Lesson for the model: weight defender *leverage and distribution* over
raw depth when calling the shell; count detached eligibles carefully
before naming the formation.

Play 3 correction — his read: **49ers in empty, 3x2; defense showing
maybe Cover 4, depending on how the bottom safety moves.** Machine had
said: shotgun 2x2, Cover 3 (low-medium confidence).

Machine error analysis: missed the 5th detached eligible (empty, not
2x2) and committed to Cover 3 off a single-high guess when the shell was
2-high. His hedge is the correct model of the problem: pre-snap shell
does not equal coverage — 2-high could be Cover 2, Cover 4, Cover 6, or a
post-snap rotation. The bottom safety's movement is the tell.

ARCHITECTURE LESSON (from his read): the analyzer should output
coverage *probabilities* pre-snap ("2-high shell: Cover 4 40%, Cover 2
35%, rotation to 1-high 25%") and *confirm post-snap* via safety
movement. Never lock a coverage call on a frozen pre-snap frame.

## Lessons

- Single frozen frames are genuinely hard; real analysts scrub. Baseline
  honesty matters more than inflated scores.
- For the labeled track, need breakdowns where the analyst explicitly IDs
  coverage (e.g. JT O'Sullivan's QB School — he names it every play).
- Next: run the same protocol on a QB School All-22 video for true
  labeled scoring.
