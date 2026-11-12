# Phrase Data and Preparation

## Content Sources

The initial dataset is based on English proverbial phrases, sayings, and expressions. Candidate sources are:

- [Wikipedia — List of proverbial phrases](https://en.wikipedia.org/wiki/List_of_proverbial_phrases)
- [The Phrase Finder — Phrases and Sayings](https://www.phrases.org.uk/meanings/phrases-and-sayings-list.html)

Source attribution and any available source context must be retained for each phrase. If a source does not provide a useful explanation, additional research should be performed. If reliable context cannot be found, a coherent explanation or hint may be authored from the phrase's meaning.

## Preparation Before Gameplay

Every phrase must be fully prepared before it becomes eligible for selection. The active game engine must not research, normalize, transliterate, or generate hints while a game is running.

Preparation includes:

1. Store the original phrase and source metadata.
2. Produce the canonical normalized phrase used by gameplay.
3. Convert numbers in the phrase into their English word forms before normalization (for example, `2` becomes `two`).
4. Process word boundaries and playable character positions.
5. Preserve punctuation and other display characters while marking them as non-playable.
6. Prepare explanatory context and exactly three complete hints.
7. Calculate and store the individual phrase difficulty.
8. Validate the prepared record before making it available to the game.

## Canonical Character Normalization

The stored gameplay phrase is normalized to ASCII letters A–Z, plus spaces and preserved display punctuation/special characters.

- Diacritics are removed or transliterated to their base ASCII letter: `é` → `E`, `ü` → `U`, `ñ` → `N`.
- Characters with standard multi-letter transliterations are expanded where appropriate: `ß` → `SS`.
- Case is normalized consistently; gameplay matching is case-insensitive.
- Numbers are converted to words before character normalization.
- Spaces define word boundaries.
- Punctuation and special characters remain visible but are not playable.

The exact Unicode transliteration library or mapping is an implementation decision. The required outcome is a deterministic normalized representation suitable for A–Z chat input.

## Viewer Input Is Not Normalized

The phrase is normalized during preparation; viewer input is not transliterated. Only ASCII A–Z and spaces are accepted after the leading slash. For example, a phrase containing `Ü` is stored with `U`, and the viewer must submit `/u`, not `/ü`.

Inputs with accented characters, numbers, punctuation, or symbols are rejected rather than silently corrected.

## Hints and Context

Every phrase has exactly three prepared hints. Hints should be complete, useful, and varied in type where the source material permits. Suitable hint dimensions include:

- Meaning or interpretation.
- Origin or historical background.
- Context in which the expression is used.
- Typical usage or an example situation.
- Synonyms or a related expression.

The three hints do not have to use three fixed categories, but they should not be redundant. Each phrase also stores its explanatory context and source information.

## Phrase Difficulty

Each phrase receives an individual difficulty label: Easy, Medium, or Hard. It is calculated during preparation/import and stored in SQLite so that the game does not calculate it during a round.

Potential inputs to the difficulty calculation include playable character count, word count, phrase length, number of unique letters, and letter frequency. The exact formula and thresholds have not yet been fixed and must be chosen during implementation planning.

Difficulty is informational only. It must not bias random selection, alter scoring, change hint timers, or modify any gameplay rule.

## Random Selection and Reuse

Phrase selection occurs during intermission, never during an active round.

1. Select the main phrase randomly from the currently available phrases.
2. Randomly choose the number of additional phrases using configurable inclusive minimum and maximum values. Initial values: minimum 2, maximum 5.
3. Select the additional phrases randomly from the available pool.
4. Build the crossings and calculate overall game difficulty only after the complete phrase set has been selected.

Selection is random and is not adjusted to force a target difficulty. If fewer phrases are available than the randomly selected number of additional phrases requires, the game may select as many as are available; the game difficulty is then calculated from the actual selected set.

Each selected phrase records its selection timestamp and becomes unavailable for the configured `phrase_reuse_delay_hours` within the current stream. The delay begins when the phrase is selected, not when the round ends.

When a new stream starts, all phrases become available again regardless of when they were selected in the previous stream. Reuse history is therefore scoped to a stream and must not prevent selection in a later stream.

## SQLite Responsibilities

SQLite is the persistent store for:

- Phrase records and normalized content.
- Original content and source attribution.
- Explanatory context and exactly three hints per phrase.
- Individual phrase difficulty and the metrics used to calculate it.
- Phrase selection timestamps and within-stream reuse state.
- Game and stream aggregate metrics.
- Configuration values that are intended to be persisted.

Current phrase state during a live round — revealed letters, active hints, countdowns, and current player scores — is transient runtime state and is not required to be persisted as a recoverable game session.

## Import Tooling

Phrase collection and phrase preparation are separate concerns. External research or extraction may produce structured input for a Rust preparation/import command. A dedicated CLI mode such as `import` or `prepare` is a preferred candidate because it can reuse the game's normalization, validation, difficulty, and database logic.

The exact ingestion format, extraction tool, command shape, duplicate policy, and update semantics are not yet fixed. They should be evaluated and specified in the implementation phase. Regardless of the chosen transport format, only validated, fully prepared records may enter the playable SQLite dataset.