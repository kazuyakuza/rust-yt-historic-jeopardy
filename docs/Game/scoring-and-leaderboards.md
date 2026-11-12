# Scoring and Leaderboards

## Player Identity

Player identity is based on the stable YouTube user ID, not the displayed channel name. The ID is retained for the entire stream so that a viewer's game points and stream victories remain associated with the same account if their display name changes.

The stream owner can be configured as eligible or ineligible to participate. When excluded, the streamer's messages may still be received and count as audience activity, but they do not participate in game scoring or leaderboards.

## Base Letter Points

Scoring is based on newly revealed occurrences only:

| Character category | Points per newly revealed occurrence |
| --- | ---: |
| Vowels (A, E, I, O, U) | 1 |
| Consonants (B–Z, excluding vowels) | 3 |

A letter guess reveals every occurrence of that letter in the relevant active phrase or phrases. Points are awarded for each newly revealed occurrence. Previously revealed letters never score again.

## Word and Phrase Bonuses

- Completing a word awards a word-completion bonus.
- Completing a phrase awards a phrase-completion bonus.
- If one input completes multiple words or phrases, all applicable bonuses are awarded.
- A bonus is awarded only when that input causes the relevant word or phrase to become complete.

Submitting a word or a complete phrase scores only the letters that were still hidden before that input, plus any completion bonuses caused by the input. Already revealed letters never score again.

Because a letter guess reveals all occurrences of that letter in a phrase, the same word cannot contain one already-revealed occurrence and another still-hidden occurrence of that same letter.

The exact numeric values of the word-completion and phrase-completion bonuses remain a balancing decision and must be fixed before scoring implementation. They must be represented consistently so tests can inspect the scoring breakdown.

## Main-Phrase Multiplier

During Phase 1, the main-phrase multiplier is determined by the number of hints already revealed, as specified in [Gameplay Rules](gameplay-rules.md): ×2.0 before any hint, ×1.5 when no more than half the hints have been shown, and ×1.0 after more than half have been shown. The multiplied score is rounded upward.

Phase 2 has no hint-based multiplier. Additional-phrase scoring uses the base letter points and completion bonuses.

## Inputs That Affect Multiple Phrases

An input may reveal letters in more than one active phrase. All newly revealed occurrences and all completion bonuses caused across those phrases are summed for the submitting player.

In Phase 2, one letter input is applied to every additional phrase simultaneously. A word or phrase guess is evaluated against the active additional phrases according to the same missing-letters-only rule.

## Per-Game Leaderboard

The game leaderboard ranks players by total points accumulated during that game.

- The highest total determines first place.
- If two or more players tie for the highest total, all are co-winners.
- There is no time-based or first-to-submit tiebreaker.
- Completing the main phrase or an additional phrase does not independently determine the winner.

Every player tied for first receives one victory point in the stream leaderboard.

## Stream Leaderboard

The stream leaderboard accumulates victory points earned across games in the current stream.

Top positions are grouped by equal victory-point totals:

- All players with the highest total share first place.
- The next lower total defines second place, and the next lower total defines third place.
- Players tied at any of these positions share that position.
- For positions 4 through 10, players with fewer victories than the third-place group are ordered alphabetically.
- The displayed list may contain more than ten players when a tie in the top three expands the number of entries. Do not truncate a tied top-three group.

The top three groups should be visually emphasized, with names listed beneath or within each group. Remaining entries can use a more compact presentation.

Stream victory totals are scoped to the current stream. Historical player and stream aggregates may be stored for metrics, but a new stream starts a new stream leaderboard.