# Gameplay Rules

## Phrase Presentation

- Every phrase is stored in a normalized form before it can enter a game.
- Each playable letter position is initially hidden.
- Word boundaries remain visible as spaces.
- Punctuation and non-playable special characters are displayed from the beginning and are never guessed.
- A letter guess reveals every occurrence of that letter in the relevant phrase, not just one occurrence.
- Letter matching is case-insensitive.

## Input Grammar

Game inputs must be sent as YouTube Live Chat messages beginning with `/`.

Accepted forms:

- `/a` — one letter.
- `/hello` — one word.
- `/a stitch in time` — a phrase or multi-word guess.

After the slash, only ASCII letters A–Z and ordinary spaces are accepted. Uppercase and lowercase are equivalent. Inputs containing digits, accented characters, punctuation, symbols, tabs, or other characters are rejected. The input is not transliterated or corrected: viewers must type the normalized A–Z form.

Messages without the leading slash are not game inputs. A message whose input length exceeds the longest phrase currently available to discover is discarded before game processing. Repeated guesses that cannot reveal anything and award no points are ignored.

Inputs are processed sequentially in FIFO order, preserving the order in which the application receives them. The game retains the submitting YouTube user ID so that a viewer's score remains associated with the same account throughout the stream.

Every received chat message, including invalid or non-game messages, still counts as audience activity for stream-lifecycle purposes.

## Phase 1 — Main Phrase

The main phrase uses all three hints prepared for that phrase.

- Hints are revealed one at a time according to a configurable countdown.
- A valid input that produces points resets the countdown for the next hint.
- An invalid input or a valid-looking input that produces no points does not reset the countdown.
- When the countdown expires, the next unrevealed hint is shown.
- Once all three hints have been shown, no further hint is available.

The score multiplier depends on how many main-phrase hints have been shown when the input is scored:

| Hints already shown | Multiplier |
| ---: | ---: |
| 0 | ×2.0 |
| Up to half of the total hints | ×1.5 |
| More than half | ×1.0 |

With exactly three hints, this means zero hints shown gives ×2, one hint shown gives ×1.5, and two or three hints shown gives ×1. The multiplied result is rounded upward. The multiplier applies only to main-phrase scoring; additional phrases do not use hint-based multipliers.

## Transition to Phase 2

Completing the main phrase immediately ends Phase 1 and activates Phase 2. This transition does not declare a game winner and does not award a victory independently of the leaderboard.

## Phase 2 — Additional Phrases

- Every additional phrase has exactly three prepared hints in the database.
- One of those three hints is selected randomly for each additional phrase.
- The selected hint is displayed immediately when Phase 2 begins.
- All additional phrases become active simultaneously.
- There is no hint countdown in this phase.
- Additional-phrase hints do not reset timers, change scoring multipliers, or otherwise affect points.
- A letter input is applied to every additional phrase, revealing all occurrences in each phrase.

Each additional phrase intersects the main phrase using one shared letter. The first occurrence of the shared letter in the additional phrase is the crossing position. All occurrences of that letter in the additional phrase are revealed when the letter is guessed. Additional phrases never intersect one another.

The exact algorithm for choosing which eligible shared letter to use when constructing a crossing is an implementation detail that must preserve these visible rules.

## Timing and YouTube Latency

YouTube Live has substantial end-to-end latency. The current operator-observed baseline is approximately 30–35 seconds between a viewer sending a message and seeing the resulting frame. This is an observation, not a platform guarantee, and may vary.

For countdowns that viewers must be able to act within, the application should first publish the visual state that starts or displays the countdown, then wait for a configurable display-to-logic delay before starting the logical timer. The initial proposed delay is 15 seconds. It must be configurable and applied selectively to relevant countdowns, not blindly to every timer.

Timers use a monotonic clock. Game simulation and timer progression must not depend on the renderer's frame rate.

## Input Handling Outside Active Play

When no game is active, chat messages are not passed to the game rules and cannot score. They may still count as audience activity and affect the stream lifecycle.