# Technical Integration

## Core Reuse

This repository contains the reusable Rust YouTube streaming Core. The game is an application-level extension of that Core, not a replacement for its transport and media infrastructure.

The existing Core documentation remains authoritative for its implemented behavior, including OAuth, broadcast creation, Live Chat server streaming, the frame pipeline, renderer interface, FFmpeg supervision, diagnostics, shutdown, and deployment. The existing [Game Foundation handoff](../game-foundation.md) documents the actual Core module map and extension seams.

Game implementation should reuse the Core's tested pipeline and extend it at the documented boundaries. Do not duplicate YouTube API, OAuth, chat transport, RTMPS, FFmpeg, or frame-pacing responsibilities inside game logic.

## Separation of Responsibilities

The game should preserve these boundaries:

- **Transport:** receives YouTube messages and exposes application-level chat message data. Transport messages are not automatically game commands.
- **Input parsing and validation:** converts chat text into accepted game inputs according to [Gameplay Rules](gameplay-rules.md).
- **Game rules:** owns phrase state, revealed letters, hints, timers, scoring, phases, and player/game state.
- **Persistence:** owns the prepared phrase dataset, source metadata, hints, difficulty, reuse state, configuration, and aggregate metrics.
- **Render projection:** derives a presentation-only view of authoritative game state.
- **Renderer:** draws the render projection through the Core's renderer interface.
- **Streaming:** remains owned by the existing Core and FFmpeg pipeline.

Game state must be authoritative and independent of renderer implementation. Rendering must not mutate game state. Game simulation and timers must not be coupled to the output frame rate.

## Core Extension Seams

The existing Core handoff identifies these relevant seams:

- `MessageSource` for supplying messages to the application pipeline.
- `Renderer` and `RenderInput` for rendering game presentation.
- The deterministic local demo source as a pattern for offline testing.
- The existing validated configuration layer for new game settings.

Use the current contracts in [Game Foundation](../game-foundation.md) and inspect the source and test rosters before changing any Core module. The chat worker, bounded channel, frame loop, FFmpeg argument construction, credential handling, and shutdown contracts should remain unchanged unless a separate, explicit architecture decision requires otherwise.

## Persistence Model

SQLite stores durable game content and historical information. At minimum, the persistent model must cover:

- Phrases, original text, normalized text, and source attribution.
- Explanatory context and exactly three prepared hints per phrase.
- Individual phrase difficulty and the features used to derive it.
- Per-stream phrase selection timestamps and reuse availability.
- Game and stream aggregate metrics.
- Historical game durations used to estimate future round duration.

Current-round state is transient: revealed letters, visible hints, countdowns, active phrase crossings, current points, input queue, and per-input scoring breakdown. The per-input breakdown should be available to tests and diagnostics while processing, but it does not need to be persisted after the game.

At stream start, initialize a new stream scope: clear stream victory totals and make every phrase available. Historical aggregates remain intact. Track, where applicable, player counts, valid-input counts, average game duration, total game count, average games per stream, players per stream, and other balance/usage metrics.

## Configuration

Game behavior must be configurable where the design identifies variable timing, capacity, or policy. Candidate settings include:

- Minimum and maximum number of additional phrases (initially 2 and 5).
- Phrase reuse delay in hours.
- Main hint interval and other gameplay countdowns.
- Display-to-logic delay for viewer-visible countdowns (initially 15 seconds).
- Short start countdown after audience activity.
- No-player/no-participation timeout during a game.
- Initial chat inactivity interval before viewer lookup.
- Viewer lookup interval after a positive count.
- No-audience timeout after a zero count.
- Viewer-query retry interval and maximum consecutive errors (initially five minutes and three errors).
- Maximum stream duration, end-warning countdown, final-results display duration, and safety margin.
- Streamer participation eligibility.
- Scoring bonuses and other balance values once finalized.

Configuration names, nesting, and exact defaults must be settled during implementation planning and reflected in the typed configuration validation and example configuration.

## Determinism and Testing

Game rules must be deterministic and testable without YouTube, OAuth, a real broadcast, or FFmpeg. Use injected clocks and deterministic random sources/seeds where practical. Tests must not sleep in real time.

At minimum, test:

- Input grammar, normalization boundaries, and rejected characters.
- FIFO processing and stable player identity.
- Letter reveal behavior across repeated occurrences and multiple phrases.
- Word/phrase guesses scoring only previously hidden letters.
- Word and phrase completion bonuses, including multiple completions in one input.
- Hint countdown progression and reset behavior.
- Main-phrase multipliers and upward rounding.
- Main-to-additional phase transition and phrase crossings.
- Random phrase selection constraints, reuse delay, and new-stream reset.
- Tied game winners and stream leaderboard ranking.
- Chat-first audience detection, viewer fallback, zero-viewer recheck, API errors, retry limits, and chat-activity cancellation.
- Stream-duration admission and forced game/stream closure.

Use the Core's local demo/testing patterns to drive game behavior offline. Preserve the Core's own tests and contracts.

## Operational Latency

The currently observed YouTube chat-to-viewer delay is approximately 30–35 seconds. This is an operator observation and can vary; it is not a guaranteed platform constant.

The configurable display-to-logic delay is a deliberately simple mitigation for countdown visibility. It does not make YouTube Live a synchronized real-time transport. The stream description and viewer instructions should explain that messages and displayed results may be delayed.

## Phrase Collection and Import Workflow

Phrase acquisition and SQLite import are separate development tools with a JSON interchange contract:

- A standalone JavaScript subproject running on Node.js collects phrase candidates from the documented sources, extracts source attribution and available context, researches or authors missing context, prepares exactly three hints per phrase, and writes structured JSON files.
- The Node.js collector lives inside the repository but outside the Rust game's `src/` tree. It is not part of the live game runtime and does not write directly to SQLite.
- A dedicated Rust executable mode or CLI command consumes the JSON files, validates their structure and content, applies or verifies canonical phrase normalization, calculates derived fields such as individual difficulty, and imports valid records through the Rust persistence layer.
- The JSON interchange contract must be versioned and documented so the collector and importer can evolve independently without silently breaking compatibility.

The collector's source-specific extraction approach, enrichment method, JSON schema details, and duplicate/update semantics are implementation decisions to resolve in the corresponding phases. The collector and importer phases are listed in the [Game Development Roadmap](development-roadmap.md).

## Implementation Constraint

This documentation defines intended game behavior and boundaries. It does not authorize changes to existing Core behavior by itself. Any Core change must be separately justified, planned, tested, and documented without weakening the frozen contracts.