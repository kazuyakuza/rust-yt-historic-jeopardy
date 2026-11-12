# Decisions to Finalize Before Implementation

This document distinguishes settled game behavior from values and algorithms that were intentionally left open. It prevents proposed examples from being mistaken for final constants.

## Settled Rules

- The main phrase is selected randomly first.
- The number of additional phrases is then selected randomly from an inclusive, configurable range. Initial range: 2–5.
- Additional phrases are selected randomly after the count is chosen. Game difficulty is calculated only after the complete selected set is known.
- Phrase selection is not biased by difficulty.
- Every phrase has exactly three prepared hints.
- Viewer input uses `/` followed only by ASCII A–Z letters and spaces. Inputs are not transliterated.
- Letter guesses reveal every occurrence of that letter in each active phrase.
- Main-phrase hint multipliers are ×2.0, ×1.5, and ×1.0 according to the number of hints shown; the result rounds upward.
- Vowels score 1 point per newly revealed occurrence; consonants score 3.
- Word and phrase completion bonuses exist, but their numeric values have not been fixed.
- Game winners are determined by points. Every first-place tie receives one stream victory point.
- Phrase reuse delay is scoped to one stream. A new stream makes every phrase available again.
- Any received chat message is audience activity; only valid scoring inputs are game participation.
- Viewer count is checked only after chat inactivity. Positive counts lead to another delayed check; zero starts a no-audience timeout followed by a final verification.
- Viewer API errors or missing values are not treated as zero. Repeated consecutive failures may trigger shutdown; the initial maximum is three.

## Numeric Values Still to Set

| Setting | Current proposal or status |
| --- | --- |
| Additional phrase count | Configurable minimum 2, maximum 5 |
| Main hint countdown | Configurable; exact initial interval not fixed |
| Display-to-logic delay | Configurable; initial proposal 15 seconds |
| Audience-start countdown | Configurable; initial example 2–3 seconds |
| No-participation timeout during a game | Configurable; initial example 5 minutes |
| Initial chat inactivity before viewer lookup | Configurable; initial proposal 5 minutes |
| Positive-viewer recheck interval | Configurable; initial proposal 5 minutes (one minute or longer is acceptable) |
| No-audience timeout after a zero count | Configurable; exact value not fixed |
| Viewer API retry interval | Configurable; initial proposal 5 minutes |
| Maximum consecutive viewer-query errors | Configurable; initial proposal 3 |
| Phrase reuse delay | Configurable in hours; 5 hours was an example, not a final default |
| Maximum stream duration | Configurable; exact value not fixed |
| Stream-ending warning countdown | Configurable; exact value not fixed |
| Final-results display duration | Configurable; exact value not fixed |
| Safety margin before admitting another game | Configurable; exact value not fixed |

## Algorithms and Policies Still to Set

1. **Scoring bonuses:** choose the numeric word-completion and phrase-completion bonuses. Also confirm explicitly whether the main-phrase hint multiplier applies to completion bonuses as well as letter points.
2. **Phrase difficulty:** define the individual phrase difficulty formula, feature weights, and Easy/Medium/Hard thresholds.
3. **Game difficulty:** define how the selected phrases' individual difficulties and aggregate features are combined into the game's Easy/Medium/Hard label.
4. **Phrase crossings:** define how to choose the shared letter when a main/additional phrase pair has multiple possible common letters. The visible rule remains one crossing per additional phrase, at the first occurrence in that additional phrase, with all occurrences revealed by a guess.
5. **Import workflow:** choose the phrase extraction/research tool, structured interchange format (JSON is one candidate), Rust CLI command, duplicate handling, and update semantics.
6. **Insufficient phrase inventory:** settle exact behavior when the random additional-phrase count exceeds the number of eligible phrases remaining in the stream. The current design allows the selected set to contain fewer extras and calculates difficulty from the actual set.
7. **No-participation game ending:** define whether an automatically ended game produces final standings and stream victory points, and how to handle a game with no scoring inputs.
8. **Streamer participation:** choose the default for whether the stream owner's YouTube account is eligible to score.
9. **Input length measurement:** define whether the maximum-input-length guard counts spaces in the normalized input and phrase length. The guard itself is settled: discard inputs longer than the longest phrase still available to discover.
10. **Visual placement:** finalize the exact placement of additional-phrase difficulty labels (beside the hint or beneath the phrase) and the overall game difficulty indicator.

## Implementation Planning Guidance

These decisions do not block documenting the concept. They should be resolved in the relevant implementation plan before the affected behavior is coded. Configurable values must be represented in the typed configuration schema and validated; they should not become scattered hard-coded constants.