# Game Documentation

This directory contains the product and gameplay specification for the YouTube chat-controlled phrase guessing game, together with its implementation roadmap.

The repository's existing root-level documentation describes the reusable YouTube streaming Core. Those documents remain unchanged. The Core integration boundary is documented in [Game Foundation](../game-foundation.md).

## Documents

- [Game Development Roadmap](development-roadmap.md) — all implementation phases, their scope, and the corresponding TODO files.
- [Game Design](game-design.md) — concept, audience experience, visual direction, and overall round structure.
- [Gameplay Rules](gameplay-rules.md) — phrase presentation, input grammar, phases, hints, and game timing.
- [Phrase Data and Preparation](phrase-data.md) — source material, normalization, hints, difficulty, SQLite, and import preparation.
- [Scoring and Leaderboards](scoring-and-leaderboards.md) — point calculation, bonuses, player identity, and stream standings.
- [Stream Lifecycle and Audience](stream-lifecycle.md) — stream states, intermissions, audience detection, viewer fallback, and shutdown.
- [Technical Integration](technical-integration.md) — Core reuse, module boundaries, persistence responsibilities, testing, and operational constraints.
- [Decisions to Finalize](decisions-to-finalize.md) — settled rules versus numeric values and algorithms to lock before implementation.

## Source of Truth

The game-specific requirements in this directory define the intended game behavior. The Project Brief at [`.agent/project-info/brief.md`](../../.agent/project-info/brief.md) provides the project-level summary and links here for detail.

The [Game Development Roadmap](development-roadmap.md) defines the progressive implementation sequence. Each phase links to its corresponding actionable TODO file under `.agent/todos/20261112/`. The TODO files should reference this documentation rather than duplicate detailed game rules or technical contracts.

When implementation-specific technical contracts of the existing streaming Core are involved, retain the existing Core documentation as the source of truth for those contracts. Game rules must not be used as a reason to rewrite or weaken the Core.
