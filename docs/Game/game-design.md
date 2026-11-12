# Game Design

## Concept

The project is a Rust-based, YouTube Live chat-controlled phrase guessing game. Viewers cooperate and compete by submitting guesses in the live chat to reveal hidden English phrases, initially drawn from proverbial phrases, sayings, and historically interesting expressions.

The core mechanic is inspired by Hangman, but guesses are not limited to individual letters: a viewer may submit a letter, a word, or a complete phrase. The game is designed around YouTube Live's substantial end-to-end latency, so it is turn-based and tolerant of delayed inputs rather than requiring frame-accurate reactions.

## Audience Experience

- The stream remains active while waiting for an audience, subject to the configured stream lifetime and audience-absence rules.
- Viewers submit guesses through ordinary YouTube Live Chat messages.
- Valid guesses reveal letters across the active phrase set and award points to the submitting viewer.
- The main phrase is presented first. Solving it unlocks a second phase with several related additional phrases.
- A per-game leaderboard identifies the highest-scoring viewer or tied viewers.
- A stream leaderboard accumulates game victories across the current stream.

All incoming chat messages count as evidence of audience activity, even when their contents are not valid game inputs. Only valid, scoring guesses count as game participation.

## Round Structure

Each game contains one main phrase and a randomly selected number of additional phrases.

1. Select one available phrase at random as the main phrase.
2. Randomly choose the number of additional phrases, using configurable inclusive minimum and maximum values. The initial proposed range is 2–5.
3. Select that many additional phrases randomly from the available phrase pool.
4. Build the phrase connections and calculate the overall game difficulty from the complete selected set.
5. Play the main phrase phase, followed by the additional-phrases phase.
6. Calculate the game leaderboard, award stream victory points to all first-place ties, and proceed to the next intermission or stream ending.

Difficulty is descriptive information for viewers. It does not influence phrase selection, scoring, timers, or game rules.

## Two Gameplay Phases

### Phase 1 — Main Phrase

The main phrase is displayed horizontally with its words and character positions visible. Playable letters are hidden until guessed. Spaces and punctuation are visible from the beginning. Three phrase-specific hints are revealed progressively over time.

### Phase 2 — Additional Phrases

Additional phrases are displayed vertically. Each additional phrase connects to the main phrase through exactly one shared letter; additional phrases do not connect to one another. One randomly selected hint for each additional phrase is displayed immediately when this phase begins.

All additional phrases are active simultaneously. A valid letter guess is applied to every active additional phrase, revealing all occurrences of that letter in each phrase.

## Visual Direction

The presentation should be simple, readable, and functional rather than animation-heavy or graphically elaborate.

- Dark background and high-contrast, clearly legible typography.
- Main phrase arranged horizontally; additional phrases arranged vertically.
- Clear separation between phrase content, hints, difficulty indicators, instructions, and leaderboards.
- Main-phrase difficulty shown near its hints.
- Additional-phrase difficulty shown near the corresponding hint or phrase.
- Overall game difficulty shown near the instructions, status, or leaderboard.
- Initial development may use a small logical resolution such as 320×180. The intended output should scale to 1920×1080 and potentially 4K without changing game rules.
- Complex animation is not a requirement.

## Game Outcome

Completing the main phrase only advances the game to Phase 2. Completing a phrase does not automatically make its solver the winner. The winner is determined exclusively by the game leaderboard's total points. If multiple viewers tie for first place, all of them receive one stream victory point.

## Initial Content Scope

The initial phrase collection is English-language proverbial phrases and sayings. Candidate source collections include:

- [Wikipedia — List of proverbial phrases](https://en.wikipedia.org/wiki/List_of_proverbial_phrases)
- [The Phrase Finder — Phrases and Sayings](https://www.phrases.org.uk/meanings/phrases-and-sayings-list.html)

Each phrase must have its source, explanatory context, exactly three prepared hints, and a stored difficulty before it can be selected for a game.

## Explicit Non-Goals

- Real-time action mechanics that require viewers to react within a few seconds.
- A crossword in which additional phrases connect to each other.
- Selecting phrases according to the game's difficulty label.
- Awarding victory merely for completing the main phrase.
- Requiring viewers to send individual letters only.
- Complex animation or a high-complexity visual design.
- Processing or researching phrase content during an active game.