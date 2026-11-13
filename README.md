# YouTube Stream Historic Jeopardy Game

A Rust-based YouTube Live game in which viewers participate through YouTube Live Chat. The first planned prototype, presented in-game as **Historic Jeopardy**, is a phrase-discovery game: players reveal letters and solve English proverbial phrases while competing for points on live and stream-wide leaderboards.

The project combines an existing, working YouTube streaming Core with a separately developed game layer. The Core already handles YouTube authentication, broadcast setup, Live Chat ingestion, frame rendering, and FFmpeg-based video streaming. The game itself is not implemented yet.

## Project Status

<badge color="success">Core implemented</badge> <badge color="info">Game design and roadmap defined</badge>

- **Streaming Core:** The reusable Rust foundation is implemented, including OAuth authentication, local demo recording, YouTube Live broadcast operation, Live Chat integration through the official server-streaming API, rendering, and FFmpeg delivery.
- **Game definition:** The first game's concept, gameplay rules, phrase data requirements, scoring, leaderboards, and stream lifecycle have been documented.
- **Development plan:** Implementation is divided into 22 progressive phases. Each phase has its own TODO file, with testing and documentation included alongside implementation work.
- **Game implementation:** Not started. The repository currently provides the Core and the agreed specifications and implementation plan.

The current Core capabilities and their operational details are documented in [Core README](docs/core-readme.md). The [Game Foundation Handoff](docs/game-foundation.md) explains how the existing Core is intended to be reused and extended.

## The Game

The initial prototype is a 2D phrase-guessing game inspired by word-reveal games. A main phrase is shown as hidden letter positions, and viewers submit guesses through chat commands. Correct inputs reveal matching letters or words and award points. Three hints are progressively revealed for the main phrase; after it is solved, additional phrases connect vertically to it through shared letters and are played together.

Players compete through a round leaderboard, while round winners accumulate victories in a stream leaderboard. The stream runs successive rounds, handles periods without audience activity, and presents final standings when it ends.

This is intentionally a modest first game: its purpose is to validate the experience of playing a game through YouTube Live Chat and the integration of game state with the existing streaming pipeline. More elaborate game concepts may follow once this prototype has been exercised.

The detailed and authoritative game requirements are in the [Project Brief](.agent/project-info/brief.md) and the specifications under [docs/Game/](docs/Game/README.md). The README is only a project-level overview.

## Technology and Foundation

- **Rust** is the primary language for the application, game engine, phrase preparation/import, and runtime integration.
- **YouTube Data API and Live Chat streaming** provide broadcast management and viewer input.
- **OAuth 2.0** provides YouTube authorization.
- **FFmpeg** is an external process used for video encoding and delivery to YouTube Live.
- **SQLite** is planned as the persistent store for prepared phrases and historical game/stream metrics.
- **Node.js (JavaScript)** is planned for a standalone phrase-collection subproject. It will gather phrase material from the documented sources and produce structured JSON files; the Rust importer will validate and load those records into SQLite.

The game layer is intended to remain independent of YouTube API and FFmpeg implementation details. It will integrate through the Core's existing application, message-source, and renderer boundaries rather than replacing the streaming pipeline.

See [Core README](docs/core-readme.md) for the implemented Core stack, commands, configuration, authentication, build workflows, and operational behavior.

## Development Roadmap

The implementation sequence is defined in the [Game Development Roadmap](docs/Game/development-roadmap.md). It contains all 22 phases and links each phase to its corresponding TODO file under [`.agent/todos/20261112/`](.agent/todos/20261112/).

Development proceeds incrementally from project and architecture alignment, through phrase collection/import and the game engine, into rendering, chat input integration, stream lifecycle, metrics, and local runtime validation. Each phase includes its own relevant tests and documentation; there is no separate final testing or documentation-only phase.

The project is currently at the planning-to-implementation handoff: the Core is the starting point, and the next step is to execute Phase 0 using the repository's agent workflow.

## Project Documentation

- [Project Brief](.agent/project-info/brief.md) — project-level requirements and source of truth for the overall game.
- [Game Documentation Index](docs/Game/README.md) — index of all game-specific specifications.
- [Game Development Roadmap](docs/Game/development-roadmap.md) — phased implementation plan and TODO references.
- [Game Foundation Handoff](docs/game-foundation.md) — existing Core architecture, extension points, and integration constraints.
- [Core README](docs/core-readme.md) — preserved Core overview and operational documentation.
- [Project Info](.agent/project-info/) — product, context, architecture, and technology information.
- [Agent Instructions](AGENTS.md) — mandatory instructions for AI agents working in this repository.

## Working with AI Agents

All AI agents must read and follow [AGENTS.md](AGENTS.md) before making changes. The repository's Critical Workflow defines how TODO files are analyzed, planned, implemented, reviewed, tested, and documented.

To start the next planned phase, use the TODO workflow described in [AGENTS.md](AGENTS.md) and process the first phase file:

[`.agent/todos/20261112/20261112-todo-1.md`](.agent/todos/20261112/20261112-todo-1.md).
