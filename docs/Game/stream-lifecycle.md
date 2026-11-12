# Stream Lifecycle and Audience

## Lifecycle States

The game application uses the following high-level states:

1. `Initializing` — load and validate configuration, initialize persistent data and game services, and prepare the stream runtime.
2. `WaitingForPlayers` — the stream is active but no game is currently accepting guesses. The application waits for audience activity.
3. `Intermission` — select and reserve the next phrase set, build crossings, calculate game difficulty, and prepare the next round.
4. `Playing` — execute the main-phrase and additional-phrases phases.
5. `FinalResults` — show final game or stream standings as appropriate.
6. `Finished` — the stream lifecycle has ended.

## Waiting for the Audience

The stream should remain active while waiting for an audience, subject to its configured maximum lifetime and shutdown rules.

When the application is waiting for players, the first received chat message establishes audience activity. A configurable short countdown then precedes the next game; an initial example is 2–3 seconds. Messages received when no game is active do not become game guesses, but they still count as audience activity.

During a game, a configurable no-participation timeout may end the current game and return the application to `WaitingForPlayers`. No participation means that no valid, scoring game input has been received during the timeout; an initial example discussed for this timeout is five minutes. This timer is distinct from stream-level chat inactivity: any incoming chat message, even an invalid one, counts as audience activity and resets chat inactivity, but it does not count as game participation.

## Chat-First Audience Detection

Chat messages are the primary audience-presence signal. The application does not query the viewer counter continuously while chat activity is arriving.

- Every received chat message, valid or invalid for the game, resets chat inactivity.
- If a chat message arrives during viewer fallback or an absence countdown, cancel that fallback/countdown and return to normal chat-driven operation.
- Only when no chat message has arrived for the configured initial inactivity interval does the application query YouTube for the current viewer count.

## Viewer Counter Fallback

The viewer counter is a fallback signal, not a real-time event listener. No official push notification for changes to concurrent viewer count has been identified. Use the YouTube Data API's `concurrentViewers` value when available, and make requests only after chat inactivity has activated the fallback. See the official [Videos resource documentation](https://developers.google.com/youtube/v3/docs/videos#liveStreamingDetails) for `liveStreamingDetails.concurrentViewers` and the [YouTube push notification guide](https://developers.google.com/youtube/v3/guides/push_notifications) for the supported notification scope.

Initial proposed timing and behavior:

1. Wait five minutes after the last chat message, then query the viewer count.
2. If the count is greater than zero, keep the stream active and wait another five minutes before querying again. Continue this cycle only while chat remains inactive.
3. If the count is zero, stop the periodic positive-viewer polling and start the configurable no-audience timeout.
4. When the no-audience timeout expires, query the viewer count one more time before starting stream shutdown.
5. If the final query is still zero, begin the stream-closing process.
6. If the final query is positive, cancel the no-audience countdown and return to normal operation.
7. If chat activity arrives at any point, cancel the fallback and the no-audience countdown.

The initial five-minute polling interval is a proposed configuration default, not a hard real-time requirement. Viewer polling should remain deliberately infrequent; one minute or longer is acceptable, and five-minute intervals are the initial design.

## Viewer API Errors and Missing Data

An API error, unavailable field, or missing `concurrentViewers` value is not equivalent to zero viewers.

- On an error or unavailable counter, wait the configured retry interval (initially five minutes) and retry if chat remains inactive.
- Count consecutive viewer-query failures. The initial proposed maximum is three.
- If the maximum number of consecutive failures is reached while there is still no chat activity, initiate stream shutdown rather than retrying indefinitely.
- A successful viewer-count response resets the consecutive-error count.
- A chat message cancels the fallback process; a later period of chat inactivity starts a new fallback cycle.

The YouTube API may omit `concurrentViewers` in some circumstances. The implementation must treat that as an unavailable result, not as a numeric zero.

## Stream Duration and Round Admission

The maximum stream duration is configurable. Before starting another game, compare the remaining stream time with the expected game duration plus a configurable safety margin and the time reserved for final results and shutdown.

Use historical average game duration when enough completed games exist. When history is insufficient, use a configurable estimated average. Do not begin a new game if the remaining stream time is insufficient for the expected game and the reserved closing sequence.

If the stream is approaching its configured end while a game is active:

1. Display a warning that the game will end in a configured countdown.
2. Apply the configured display-to-logic delay where the countdown is intended to be actionable by viewers.
3. When the countdown expires, end the game even if its phrases are incomplete.
4. Determine game positions from points accumulated so far.
5. Award stream victory points to every player tied for first.
6. Display the final stream leaderboard.
7. Keep the final-results screen visible for a configurable duration.
8. Finish the stream.

If no game is active when the stream reaches its closing point, skip the game-ending steps and proceed to final results and shutdown.

## New Stream Reset

At the beginning of every new stream:

- All phrases become available again, regardless of their selection timestamps in previous streams.
- The stream leaderboard starts with zero victory points.
- Stream-scoped player/session state and inactivity/error counters are reset.
- Historical aggregate metrics remain available for duration estimates and analytics.

## Phrase Reservation Within a Stream

Phrase reuse delay is scoped to a single stream. A phrase becomes unavailable when selected and is eligible again after the configured number of hours has elapsed. The timer begins at selection time, even if the phrase's game is later interrupted. A new stream resets availability for the entire phrase pool.