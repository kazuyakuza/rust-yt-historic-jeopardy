# Game Development Roadmap

## Purpose

This document defines the progressive implementation phases for the game described in the [Project Brief](../../.agent/project-info/brief.md) and the specifications in this directory.

Each phase has a corresponding TODO file under `.agent/todos/`. The TODO files are the actionable work items for implementation agents. They follow the repository's TODO conventions and expand the phase into task-sized items without duplicating the detailed game rules or technical contracts already documented here.

Implementation agents must read the referenced specifications before planning a task. If a required decision is still open, the relevant TODO must begin by resolving that decision before implementation proceeds. Testing and documentation are part of each phase, not deferred to a final phase.

## Phase Overview

| Phase | Name | TODO |
|---:|---|---|
| 0 | Project Setup and Documentation Alignment | [20261112-todo-1.md](../../.agent/todos/20261112/20261112-todo-1.md) |
| 1 | Game Domain Foundation | [20261112-todo-2.md](../../.agent/todos/20261112/20261112-todo-2.md) |
| 2 | Persistence and Phrase Database Foundation | [20261112-todo-3.md](../../.agent/todos/20261112/20261112-todo-3.md) |
| 3 | Phrase Source Collection and JSON Generation | [20261112-todo-4.md](../../.agent/todos/20261112/20261112-todo-4.md) |
| 4 | Phrase JSON Importer and Data Validation | [20261112-todo-5.md](../../.agent/todos/20261112/20261112-todo-5.md) |
| 5 | Main and Additional Phrase Gameplay | [20261112-todo-6.md](../../.agent/todos/20261112/20261112-todo-6.md) |
| 6 | Hints and Timers | [20261112-todo-7.md](../../.agent/todos/20261112/20261112-todo-7.md) |
| 7 | Scoring System | [20261112-todo-8.md](../../.agent/todos/20261112/20261112-todo-8.md) |
| 8 | Game Leaderboard and Tie Resolution | [20261112-todo-9.md](../../.agent/todos/20261112/20261112-todo-9.md) |
| 9 | Additional Phrases Vertical Presentation | [20261112-todo-10.md](../../.agent/todos/20261112/20261112-todo-10.md) |
| 10 | Game Information and Instructions UI | [20261112-todo-11.md](../../.agent/todos/20261112/20261112-todo-11.md) |
| 11 | Game Leaderboard Presentation | [20261112-todo-12.md](../../.agent/todos/20261112/20261112-todo-12.md) |
| 12 | Output Resolution and Core Renderer Integration | [20261112-todo-13.md](../../.agent/todos/20261112/20261112-todo-13.md) |
| 13 | Chat Input Processing and Game Updates | [20261112-todo-14.md](../../.agent/todos/20261112/20261112-todo-14.md) |
| 14 | Demo Input Source | [20261112-todo-15.md](../../.agent/todos/20261112/20261112-todo-15.md) |
| 15 | Stream Leaderboard | [20261112-todo-16.md](../../.agent/todos/20261112/20261112-todo-16.md) |
| 16 | Chat Activity Detection | [20261112-todo-17.md](../../.agent/todos/20261112/20261112-todo-17.md) |
| 17 | Viewer Fallback and Audience Timeout | [20261112-todo-18.md](../../.agent/todos/20261112/20261112-todo-18.md) |
| 18 | Maximum Stream Duration and Final Results | [20261112-todo-19.md](../../.agent/todos/20261112/20261112-todo-19.md) |
| 19 | Phrase Availability Reset on New Stream | [20261112-todo-20.md](../../.agent/todos/20261112/20261112-todo-20.md) |
| 20 | Historical Metrics and Game Admission | [20261112-todo-21.md](../../.agent/todos/20261112/20261112-todo-21.md) |
| 21 | Reliability and Local Runtime Configuration | [20261112-todo-22.md](../../.agent/todos/20261112/20261112-todo-22.md) |

## Phase Details

### Phase 0 — Project Setup and Documentation Alignment

**TODO:** [20261112-todo-1.md](../../.agent/todos/20261112/20261112-todo-1.md)

Prepare the repository and agent-facing project documentation for game development.

Scope:
- Reorganize Project Info according to the requirements and decisions recorded in this TODO, while preserving the existing Core documentation and its technical contracts.
- Align project-level descriptions with the game as the product being developed.
- Review the boundaries between the game and the existing streaming Core.
- Establish the initial game source layout and document the intended ownership of modules.
- Verify that the Brief, Project Info, Game documentation, and Core handoff do not contradict one another.

This phase is preparatory. It must not implement gameplay features.

### Phase 1 — Game Domain Foundation

**TODO:** [20261112-todo-2.md](../../.agent/todos/20261112/20261112-todo-2.md)

Establish the game domain independently of persistence, rendering, YouTube, and the stream lifecycle.

Scope:
- Define the core domain concepts and their relationships.
- Define phrase, word, playable-character, hint, player, input, and game-state representations.
- Implement the canonical text and input normalization/validation primitives required by the game.
- Separate authoritative game state from presentation projections and external message types.
- Introduce deterministic seams for time and randomness where needed by domain tests.

### Phase 2 — Persistence and Phrase Database Foundation

**TODO:** [20261112-todo-3.md](../../.agent/todos/20261112/20261112-todo-3.md)

Create the durable SQLite foundation used by phrase preparation and gameplay.

Scope:
- Design and implement the phrase and source metadata schema.
- Define persistence boundaries and repositories for phrase data.
- Implement database initialization and schema migrations.
- Provide the basic data-access operations needed by later import and selection phases.
- Test schema creation, migrations, constraints, and persistence behavior.

Phrase availability selection and stream-specific reuse lifecycle are intentionally handled in Phase 19, not in this foundational database phase.

### Phase 3 — Phrase Source Collection and JSON Generation

**TODO:** [20261112-todo-4.md](../../.agent/todos/20261112/20261112-todo-4.md)

Implement a standalone data-collection subproject that researches and extracts phrase records from the agreed web sources and produces structured JSON files.

Scope:
- Create a separate Node.js/JavaScript subproject inside the repository, outside the game's Rust `src/` tree.
- Implement extraction for the sources listed in [Phrase Data and Preparation](phrase-data.md).
- Collect source attribution and available explanatory material.
- Complete missing context and hint material as required by the phrase-data specification; each phrase must be prepared with exactly three hints.
- Produce versioned/validated JSON output suitable for the Rust importer.
- Handle duplicate candidates, source failures, incomplete records, and repeatable collection runs.
- Document local setup, configuration, execution, output format, and limitations.

The collector is a data-preparation tool, not part of the live game runtime. Node.js is the selected runtime for this subproject.

### Phase 4 — Phrase JSON Importer and Data Validation

**TODO:** [20261112-todo-5.md](../../.agent/todos/20261112/20261112-todo-5.md)

Implement the Rust-side import workflow that consumes the JSON output from Phase 3.

Scope:
- Add an executable mode or CLI command for importing prepared phrase JSON.
- Parse and validate the agreed JSON contract.
- Apply or verify the canonical phrase preparation rules.
- Calculate and persist derived phrase attributes, including individual difficulty.
- Import valid records into SQLite with controlled duplicate and error behavior.
- Provide useful import summaries and actionable validation errors.

The JSON collector and Rust importer remain separate tools with a documented contract between them.

### Phase 5 — Main and Additional Phrase Gameplay

**TODO:** [20261112-todo-6.md](../../.agent/todos/20261112/20261112-todo-6.md)

Implement the main phrase and additional-phrase gameplay flow.

Scope:
- Select the main phrase and the configured random number of additional phrases.
- Build each additional phrase's single-letter intersection with the main phrase; additional phrases do not intersect one another.
- Calculate the overall game difficulty from the selected phrases.
- Implement the main phase and transition to the simultaneous additional-phrases phase.
- Implement letter revelation and evaluation of letter, word, and phrase guesses across all active phrases.

### Phase 6 — Hints and Timers

**TODO:** [20261112-todo-7.md](../../.agent/todos/20261112/20261112-todo-7.md)

Implement hint selection, visibility, countdowns, and timer transitions.

Scope:
- Implement the three-hint progression for the main phrase.
- Implement the immediate single randomly selected hint per additional phrase when their phase begins.
- Implement the configurable hint countdown and reset conditions.
- Implement the display-to-logic delay used to make viewer-visible countdowns less vulnerable to observed chat latency.
- Test timer transitions with an injected clock; tests must not depend on real sleeping.

### Phase 7 — Scoring System

**TODO:** [20261112-todo-8.md](../../.agent/todos/20261112/20261112-todo-8.md)

Implement point calculation and scoring modifiers.

Scope:
- Resolve any remaining numeric scoring decisions before implementation.
- Implement points for newly revealed vowel and consonant occurrences.
- Implement word and phrase completion bonuses.
- Implement the main-phase hint-dependent multipliers and required rounding.
- Support one input scoring across multiple words and/or phrases.
- Expose a transient per-input scoring breakdown for tests and diagnostics without requiring permanent storage.

### Phase 8 — Game Leaderboard and Tie Resolution

**TODO:** [20261112-todo-9.md](../../.agent/todos/20261112/20261112-todo-9.md)

Implement game-level standings and winner determination.

Scope:
- Track player points during a game.
- Include players once they have earned positive points.
- Provide the top-ten game leaderboard for presentation.
- Treat all players tied for the highest score as co-winners.
- Emit one stream victory for each co-winner.
- Test ties, ordering, zero-score players, and leaderboard limits.

### Phase 9 — Additional Phrases Vertical Presentation

**TODO:** [20261112-todo-10.md](../../.agent/todos/20261112/20261112-todo-10.md)

Implement the visual layout for the additional-phrases phase.

Scope:
- Present all additional phrases vertically and simultaneously.
- Show their intersections with the main phrase clearly.
- Keep the layout readable as the number and length of phrases vary.
- Validate layout behavior with representative game-state fixtures.

### Phase 10 — Game Information and Instructions UI

**TODO:** [20261112-todo-11.md](../../.agent/todos/20261112/20261112-todo-11.md)

Present the information viewers need to understand and participate.

Scope:
- Display hints and their reveal state.
- Display individual phrase difficulty and overall game difficulty in the appropriate locations.
- Display concise input instructions and current game state.
- Prepare the complete viewer-facing rules text for reuse in the YouTube video description.
- Ensure the additional-phrase hint presentation does not imply a phrase-to-hint association beyond the specified design.

### Phase 11 — Game Leaderboard Presentation

**TODO:** [20261112-todo-12.md](../../.agent/todos/20261112/20261112-todo-12.md)

Implement the visual game leaderboard.

Scope:
- Present the top-ten game standings.
- Make ranking and score information easy to scan.
- Handle tied scores without suggesting a false winner or time-based tiebreaker.
- Validate overflow and long player-name behavior.

### Phase 12 — Output Resolution and Core Renderer Integration

**TODO:** [20261112-todo-13.md](../../.agent/todos/20261112/20261112-todo-13.md)

Adapt the complete game presentation to the configured output and connect it to the existing Core renderer.

Scope:
- Adapt layout to the selected output resolution and the design constraints.
- Integrate the game render projection with the Core `Renderer` / `RenderInput` extension seam.
- Preserve the separation between game state and rendering.
- Validate frame generation and renderer integration using local fixtures and the existing Core pipeline.
- Avoid changing frozen Core transport, FFmpeg, and lifecycle behavior unless separately justified.

### Phase 13 — Chat Input Processing and Game Updates

**TODO:** [20261112-todo-14.md](../../.agent/todos/20261112/20261112-todo-14.md)

Connect incoming chat messages to the active game.

Scope:
- Adapt Core chat messages into the game input boundary.
- Preserve FIFO processing and stable YouTube user identity.
- Apply validated inputs to the active game.
- Update scores and the render projection after each accepted scoring input.
- Ensure messages received outside an active game cannot modify game state.
- Test message-to-game integration with deterministic message fixtures.

### Phase 14 — Demo Input Source

**TODO:** [20261112-todo-15.md](../../.agent/todos/20261112/20261112-todo-15.md)

Provide an offline way to play and validate the game without a real YouTube stream.

Scope:
- Implement a demo message source following the existing Core demo patterns.
- Support deterministic synthetic players and chat inputs.
- Exercise complete games and presentation transitions locally.
- Keep demo behavior independent of OAuth and live YouTube chat.
- Document how to launch and use the local demo.

### Phase 15 — Stream Leaderboard

**TODO:** [20261112-todo-16.md](../../.agent/todos/20261112/20261112-todo-16.md)

Implement stream-scoped victories and standings.

Scope:
- Accumulate one victory point for each game co-winner.
- Implement the documented tie grouping and ordering rules.
- Provide the stream leaderboard data needed during intermissions and final results.
- Keep stream-scoped standings separate from historical aggregate metrics.
- Test ties that expand the top-three groups and the resulting display list.

### Phase 16 — Chat Activity Detection

**TODO:** [20261112-todo-17.md](../../.agent/todos/20261112/20261112-todo-17.md)

Implement chat-first audience activity detection.

Scope:
- Treat every received chat message as audience activity, regardless of whether it is a valid game input.
- Distinguish audience activity from valid game participation.
- Reset or cancel pending audience-absence checks when chat activity resumes.
- Integrate activity events with the stream lifecycle without coupling them to scoring.

### Phase 17 — Viewer Fallback and Audience Timeout

**TODO:** [20261112-todo-18.md](../../.agent/todos/20261112/20261112-todo-18.md)

Implement the periodic YouTube viewer-count fallback and absence decision flow.

Scope:
- Query concurrent viewers only after the configured period without chat activity.
- When the count is positive, wait the configured interval before another query if chat remains inactive.
- When the count is zero, start the no-audience timeout and perform a final viewer-count recheck before closing.
- Cancel the process if chat activity resumes.
- Treat API errors as unknown, not as zero viewers; retry at the configured interval and apply the configured consecutive-error limit.
- Test zero, positive, error, retry, final-recheck, and chat-resumption paths with injected time and API responses.

### Phase 18 — Maximum Stream Duration and Final Results

**TODO:** [20261112-todo-19.md](../../.agent/todos/20261112/20261112-todo-19.md)

Implement the maximum stream duration and orderly finalization.

Scope:
- Enforce the configured maximum stream duration.
- Decide whether another game can start using the remaining time and the current game-duration estimate.
- Warn viewers and show a countdown when the active game must be cut short to finish the stream.
- End the active game at the deadline and determine standings from the scores at that moment.
- Present the final stream leaderboard for its configured display duration before the Core stream shutdown.

### Phase 19 — Phrase Availability Reset on New Stream

**TODO:** [20261112-todo-20.md](../../.agent/todos/20261112/20261112-todo-20.md)

Implement phrase reuse tracking within a stream and reset it between streams.

Scope:
- Record when phrases are selected for a game.
- Exclude recently selected phrases for the configured reuse interval within the same stream.
- Make phrases available again when their individual reuse interval expires.
- Make all phrases available when a new stream starts, independently of the previous stream's end time.
- Test timestamp boundaries, repeated selections, and stream initialization.

### Phase 20 — Historical Metrics and Game Admission

**TODO:** [20261112-todo-21.md](../../.agent/todos/20261112/20261112-todo-21.md)

Persist historical metrics and use duration estimates when deciding whether another game can start.

Scope:
- Persist the agreed game and stream aggregate metrics.
- Record the information needed to calculate average game duration and average games per stream.
- Define and implement the fallback duration estimate used when historical samples are insufficient.
- Calculate the expected duration of a new game from available historical data and configured estimates.
- Integrate the estimate with stream game-admission decisions.
- Keep transient per-input scoring breakdowns out of permanent storage.

### Phase 21 — Reliability and Local Runtime Configuration

**TODO:** [20261112-todo-22.md](../../.agent/todos/20261112/20261112-todo-22.md)

Harden the integrated local application and document its operation.

Scope:
- Review error propagation and recovery across the game, database, collector/import workflow, chat integration, viewer fallback, renderer, and Core lifecycle.
- Define and validate the local runtime configuration and its defaults.
- Verify behavior when external services fail or return incomplete data.
- Document local setup, configuration, execution modes, data preparation, and troubleshooting.
- Perform end-to-end validation locally, including a real YouTube test when practical.

This phase targets local development and operation, initially on Windows. VPS deployment, Ubuntu Server setup, remote credential transfer, and headless server preparation are explicitly out of scope.

## Cross-Phase Delivery Requirements

Every TODO phase must include the testing and documentation work appropriate to its scope. These are not deferred to Phase 21.

- **Testing:** add or update unit, integration, fixture-based, and/or manual validation procedures alongside the implementation. Prefer injected clocks, deterministic random sources, and simulated message/API sources over real waits or live services.
- **Documentation:** update the relevant Game or Core documentation when a contract, behavior, configuration, command, data format, or operational procedure is added or changed.
- **References:** TODO files must link to the applicable source-of-truth documents in `docs/Game/` and the relevant Core handoff/documentation. They should summarize scope, not copy detailed specifications.
- **Open decisions:** resolve a pending decision in the earliest task that depends on it, record the decision in the appropriate specification, and only then implement the affected behavior.
- **Core protection:** reuse the existing Core interfaces and preserve its documented contracts. Changes to Core behavior require explicit justification and focused tests.
- **Local-first:** all phases must remain testable locally wherever practical. A real stream is not a prerequisite for ordinary domain, gameplay, persistence, or renderer tests.
