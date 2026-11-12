# Project Brief — Rust YouTube Live Phrase Guessing Game

Document type: Product and Game Requirements Brief
Project name: Rust YouTube Live Phrase Guessing Game
Primary language: Rust
Target platform: Windows and Linux development; Ubuntu Server production
Streaming platform: YouTube Live
Persistent game data: SQLite

## 1. Project Overview

### 1.1 Purpose

Build a chat-controlled phrase guessing game that runs as a YouTube Live broadcast. Viewers interact by submitting guesses through YouTube Live Chat. The game reveals hidden letters in English phrases, awards points for newly revealed content and completions, and maintains per-game and per-stream leaderboards.

The core mechanic is inspired by Hangman, but viewers can guess individual letters, complete words, or entire phrases. The initial content focuses on proverbial phrases, sayings, and expressions, with source attribution, explanatory context, and prepared hints.

The game is an application-level extension of the existing Rust YouTube streaming Core in this repository. The Core remains responsible for OAuth, YouTube broadcast and chat transport, frame delivery, rendering interfaces, FFmpeg, and stream shutdown. Game logic must reuse those capabilities rather than duplicate them.

### 1.2 Product Goals

The game must:

1. Run as a persistent YouTube Live experience, waiting for audience activity and managing multiple games during a stream.
2. Accept validated letter, word, and phrase guesses from YouTube Live Chat.
3. Present one main phrase followed by a set of related additional phrases.
4. Reveal every occurrence of a guessed letter and award points only for newly revealed occurrences.
5. Use prepared hints, phrase difficulty labels, and game difficulty information to help viewers.
6. Maintain a game leaderboard and a stream-wide victory leaderboard.
7. Prepare and validate all phrase content before gameplay, storing it in SQLite.
8. Handle audience absence, viewer-count fallback, maximum stream duration, and final stream results.
9. Remain deterministic and testable offline, without requiring a real YouTube broadcast for game-rule tests.
10. Reuse the existing Core while keeping game rules independent of YouTube API and FFmpeg implementation details.

### 1.3 Current Repository Context

The repository currently contains the reusable Rust YouTube streaming Core and its existing documentation. The game-specific design is defined by this brief and the documents in [`docs/Game/`](../../docs/Game/README.md). The existing Core documentation, including [`docs/game-foundation.md`](../../docs/game-foundation.md), remains in place and continues to describe the Core's actual contracts and integration seams.

This brief describes the intended game. It must not be interpreted as evidence that the game mechanics have already been implemented.

## 2. Core Gameplay Concept

Each game uses one randomly selected main phrase and a randomly selected number of additional phrases.

1. Select the main phrase randomly from the phrases currently available in the stream.
2. Randomly choose the number of additional phrases. The initial inclusive range is 2–5; both bounds are configurable.
3. Select the additional phrases randomly.
4. Connect each additional phrase to the main phrase through one shared letter. Additional phrases do not connect to one another.
5. Calculate the overall game difficulty only after all phrases have been selected.
6. Play the main phrase phase, then activate all additional phrases simultaneously.

Phrase selection is random and is not adjusted according to difficulty. Difficulty is informational and does not alter selection, scoring, timers, or game rules.

## 3. Gameplay Phases

### 3.1 Main Phrase

The main phrase is displayed horizontally. Playable letters are hidden, while spaces, punctuation, and non-playable characters remain visible. The phrase has exactly three prepared hints, revealed one at a time according to a configurable countdown.

A valid input that awards points resets the countdown for the next hint. Invalid or non-scoring inputs do not reset it. The main phrase uses a hint-dependent score multiplier:

| Hints already shown | Multiplier |
| ---: | ---: |
| 0 | ×2.0 |
| Up to half of the three hints | ×1.5 |
| More than half | ×1.0 |

The multiplied score is rounded upward. With three hints, the effective sequence is ×2 before any hint, ×1.5 after one hint, and ×1 after two or three hints.

Completing the main phrase activates the additional-phrases phase. It does not itself determine the game winner.

### 3.2 Additional Phrases

Additional phrases are displayed vertically. Each intersects the main phrase through exactly one shared letter; additional phrases never intersect one another. The first occurrence of the shared letter in the additional phrase is the crossing position, and guessing that letter reveals all its occurrences in that phrase.

Each additional phrase has exactly three prepared hints. One hint is selected randomly and displayed immediately when this phase begins. There is no hint countdown or hint-based multiplier in this phase.

All additional phrases are active at the same time. An input is applied across the entire active set.

## 4. Chat Input and Validation

Game inputs must begin with `/`. Accepted forms are `/a`, `/hello`, or `/a stitch in time`.

After the slash, only ASCII letters A–Z and ordinary spaces are accepted. Matching is case-insensitive. Inputs containing numbers, accented characters, punctuation, symbols, or other characters are rejected. Viewer input is not transliterated or corrected; viewers must use the normalized A–Z form.

Messages without `/` are not game inputs. Inputs longer than the longest phrase still available to discover are discarded. Repeated guesses that reveal nothing and award no points are ignored.

Inputs are processed sequentially in FIFO order. Player identity is the stable YouTube user ID and is retained throughout the stream. Every incoming chat message, including invalid or non-game messages, counts as audience activity; only valid scoring guesses count as game participation.

When no game is active, chat messages cannot score or change game state, but they still count as audience activity.

## 5. Phrase Preparation and Difficulty

All phrase preparation happens before a phrase can be selected for gameplay. The game must not perform research, transliteration, normalization, hint generation, or difficulty calculation during a live round.

Prepared phrase data includes:

- Original phrase and source attribution.
- Canonical normalized phrase.
- Explanatory context.
- Exactly three complete hints.
- Individual difficulty label: Easy, Medium, or Hard.
- The phrase features used to calculate difficulty.
- Selection and reuse metadata.

Numbers in source phrases are converted to English words before normalization. Diacritics are transliterated to ASCII equivalents (for example, `ü` to `U` and `ñ` to `N`); characters such as `ß` may expand to multiple ASCII letters. Spaces define word boundaries. Punctuation and special characters remain visible but are not playable.

Individual phrase difficulty is calculated during import and stored in SQLite. Candidate features include playable character count, word count, phrase length, unique-letter count, and letter frequency. The exact formula and thresholds remain an implementation decision.

Game difficulty is calculated dynamically after the main phrase and additional phrases have been selected. It may consider the selected phrases' individual difficulty and aggregate characteristics such as playable characters, word count, and number of additional phrases. It is informational only.

Initial candidate sources:

- [Wikipedia — List of proverbial phrases](https://en.wikipedia.org/wiki/List_of_proverbial_phrases)
- [The Phrase Finder — Phrases and Sayings](https://www.phrases.org.uk/meanings/phrases-and-sayings-list.html)

If a source does not provide context, additional research should be performed. If useful context cannot be found, a coherent explanation or hint may be authored from the phrase's meaning.

Phrase extraction and preparation tooling is not yet locked. A Rust CLI import/prepare mode that reuses the game's normalization, validation, difficulty, and SQLite code is a preferred direction; external research may provide structured input such as JSON. The exact format and workflow must be decided during implementation planning.

## 6. Scoring and Leaderboards

### 6.1 Scoring

- Each newly revealed vowel occurrence awards 1 point.
- Each newly revealed consonant occurrence awards 3 points.
- Completing a word awards a word-completion bonus.
- Completing a phrase awards a phrase-completion bonus.
- If one input completes multiple words or phrases, all applicable bonuses are awarded.
- A word or phrase guess scores only letters that were still hidden before that input, plus completion bonuses caused by that input. Previously revealed letters never score again.

The exact numeric values of the word and phrase bonuses remain a balancing decision and must be fixed before scoring implementation. Tests should be able to inspect the scoring breakdown for each input; per-input breakdowns do not need to be persisted.

### 6.2 Game Leaderboard

Players are ranked by total points in the current game. Every player tied for first place is a co-winner. There is no time-based tiebreaker. Solving a phrase does not automatically make a player the winner.

Each first-place player receives one victory point in the stream leaderboard.

### 6.3 Stream Leaderboard

The stream leaderboard accumulates victory points across games in the current stream. Equal totals share a position. The top three positions are grouped by victory total; positions 4–10 are ordered alphabetically among players below the third-place group. Ties that expand the top three must not be truncated, so the display may contain more than ten players.

The top three groups should be visually emphasized; remaining entries may be displayed more compactly. A new stream starts with an empty stream leaderboard.

## 7. Stream and Audience Lifecycle

High-level states are `Initializing`, `WaitingForPlayers`, `Intermission`, `Playing`, `FinalResults`, and `Finished`.

The stream remains active while waiting for an audience, subject to its configured maximum duration and audience-absence rules. When audience activity first appears, a short configurable countdown (initial example: 2–3 seconds) precedes gameplay. Intermission is used to select and reserve the next phrase set, construct crossings, and calculate game difficulty.

### 7.1 Chat-First Audience Detection

Any received chat message is audience activity, regardless of whether it is a valid game input. Chat activity cancels viewer-fallback polling and any pending no-audience countdown. Viewer queries are only used after a configured period without chat messages.

### 7.2 Viewer Counter Fallback

No official push listener for concurrent viewer-count changes has been identified. Use YouTube Data API `concurrentViewers` queries as an infrequent fallback, not as a continuously polled primary signal.

Initial proposed behavior:

1. After five minutes without chat messages, query the viewer count.
2. If the count is positive, wait another five minutes and query again, provided chat remains inactive.
3. If the count is zero, stop the periodic positive-viewer checks and start a configurable no-audience timeout.
4. When that timeout expires, query again before initiating shutdown.
5. If the final count is still zero, begin closing the stream. If it is positive, cancel the countdown and return to normal operation.
6. Any chat message cancels the fallback process and pending shutdown.

An API error or missing viewer-count field is not zero. Retry after the configured interval (initial example: five minutes). After the configured maximum number of consecutive errors (initial example: three), initiate shutdown if there is still no chat activity. A successful query resets the consecutive-error count.

### 7.3 Stream Duration and Final Results

Maximum stream duration is configurable. Before starting another game, compare remaining time with the expected game duration, a safety margin, and the time reserved for final results and shutdown. Use historical average game duration when sufficient data exists; otherwise use a configurable estimate.

If the stream is nearing its end during a game, display a configurable warning countdown, then end the game even if phrases remain incomplete. Rank players by their current points, award stream victory points to all first-place ties, display the final stream leaderboard for a configurable duration, and finish the stream. If no game is active, proceed directly to final results and shutdown.

## 8. Phrase Availability and Persistence

SQLite stores phrases, normalized content, sources, context, hints, individual difficulty, selection timestamps, reuse state, aggregate metrics, and relevant persistent configuration.

A phrase becomes unavailable when selected and becomes available again after the configured `phrase_reuse_delay_hours`, measured from selection time. This reuse delay applies only within the current stream. When a new stream starts, all phrases become available again regardless of selection timestamps in previous streams.

Current-round state — revealed letters, visible hints, timers, active phrases, current scores, and input processing details — is transient runtime state. Historical aggregates such as game durations, game counts, player counts, valid-input counts, and stream metrics are retained.

## 9. Timing, Latency, and Visual Presentation

YouTube Live has significant end-to-end latency. The current operator-observed message-to-visible baseline is approximately 30–35 seconds; it is not a platform guarantee and may vary.

For viewer-visible countdowns, publish the visual countdown state and wait a configurable display-to-logic delay before starting the logical timer. The initial proposed delay is 15 seconds. Apply it selectively to relevant countdowns rather than every timer. Use a monotonic clock, and keep game timing independent of the renderer's FPS.

The visual design is intentionally simple: dark background, clear high-contrast typography, horizontal main phrase, vertical additional phrases, readable hints, difficulty labels, instructions, and leaderboards. A small logical prototype resolution such as 320×180 may be used initially, with later output at 1080p or 4K.

## 10. Core Integration and Architecture

Reuse the existing Core's tested infrastructure. The game must remain independent of YouTube API types, OAuth credentials, broadcast IDs, RTMPS URLs, and FFmpeg command details.

Relevant existing extension seams are:

- `MessageSource` for application-level message input.
- `Renderer` and `RenderInput` for rendering the game presentation.
- The deterministic local demo source as a pattern for offline game tests.
- The existing typed configuration layer for validated game settings.

Game state is authoritative. The renderer receives a derived presentation state and must not mutate gameplay. Game logic, phrase selection, scoring, and timers must be deterministic and testable with injected clocks and random sources. The frame loop's output FPS must not define the game simulation cadence.

See [`docs/game-foundation.md`](../../docs/game-foundation.md) for the Core's actual module map, data flow, frozen contracts, and extension points. See [`docs/Game/technical-integration.md`](../../docs/Game/technical-integration.md) for game-specific boundaries.

## 11. Configuration Principles

All runtime values that control timing, capacity, participation, selection range, reuse, stream duration, viewer polling, retries, or scoring balance must be configurable where appropriate.

Initial examples and proposed defaults are not hard-coded requirements unless explicitly identified as fixed gameplay rules. Exact configuration names, schema nesting, and validation belong to implementation planning.

## 12. Non-Goals and Constraints

- Do not replace or duplicate the existing YouTube streaming Core.
- Do not couple game logic to YouTube API or FFmpeg implementation details.
- Do not process or research phrase content during gameplay.
- Do not connect additional phrases to each other.
- Do not select phrases to force a target difficulty.
- Do not award victory solely for completing the main phrase.
- Do not treat every chat message as a valid game input; it is audience activity, not necessarily participation.
- Do not rely on real-time viewer-count push events that are not provided by the documented API.
- Do not require frame-accurate or low-latency reactions from viewers.
- Do not introduce complex animation as a requirement.

## 13. Detailed Game Documentation

The detailed requirements and behavior are organized as follows:

- [`docs/Game/README.md`](../../docs/Game/README.md) — documentation index and source-of-truth guidance.
- [`docs/Game/game-design.md`](../../docs/Game/game-design.md) — concept, audience experience, round structure, and visual direction.
- [`docs/Game/gameplay-rules.md`](../../docs/Game/gameplay-rules.md) — input grammar, phrase presentation, phases, hints, and timing.
- [`docs/Game/phrase-data.md`](../../docs/Game/phrase-data.md) — source material, normalization, hints, difficulty, SQLite, and import preparation.
- [`docs/Game/scoring-and-leaderboards.md`](../../docs/Game/scoring-and-leaderboards.md) — point rules, bonuses, player identity, and rankings.
- [`docs/Game/stream-lifecycle.md`](../../docs/Game/stream-lifecycle.md) — lifecycle states, audience detection, viewer fallback, stream duration, and shutdown.
- [`docs/Game/technical-integration.md`](../../docs/Game/technical-integration.md) — Core reuse, persistence, configuration, testing, and technical boundaries.

## 14. Important Note for AI Agents

This brief is the source of truth for the intended game scope and requirements. The detailed documents linked above expand the rules without changing their meaning. Existing Core documentation remains authoritative for implemented streaming infrastructure contracts.

Before implementing a game feature, read `AGENTS.md`, this brief, the relevant document in `docs/Game/`, the existing Core handoff at `docs/game-foundation.md`, and the relevant source modules and test rosters. Do not infer that a game feature exists merely because it is specified here.