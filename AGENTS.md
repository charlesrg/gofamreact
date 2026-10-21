# Go Fam React Agent Guide

## Project

Go Fam React is a small, static reaction-time game. The game is implemented in `index.html`; there is no package manager, build step, or test framework. Keep it dependency-free unless the user asks otherwise.

## Files

- `index.html` contains the game, audio, local fallback scores, and high-score submission.
- `highscores.html` displays the monthly global leaderboard. Keep its high-score table ID and score conversion consistent with `index.html`.
- `.github/workflows/trigger-site-deploy.yml` notifies the `charlesrg/gofamexplore.com` repository after pushes to this repo's `main` branch.

## Gameplay Invariants

- SPACE and canvas `pointerdown` both start or play the current round. Keep touch input responsive and do not let canvas taps scroll or zoom the page.
- Reaction timing starts when the door-opening animation begins. `performance.now()` is the timing origin; normalize `event.timeStamp` because older Safari versions may report epoch-based timestamps.
- Input before opening starts, and reactions under `MIN_REACTION_MS`, are false starts. They must not update the best time or be submitted.
- `best` is the minimum valid reaction time for the current game. Show the same value at game over and submit it when the player saves a name.
- For hits under 500 ms, the next-round deadline equals that hit's reaction time. At 500 ms or above, retain the existing halving rule.
- Reaction values are measured in milliseconds by the browser and converted to nanoseconds when saving. The API ranks higher numeric scores first, so save `MAX_SCORE - nanoseconds`; reverse that mapping for display. Keep the value nonnegative and within the API's supported integer range.

## High Scores

- The global board uses the High Score API at `https://highscore.sasagu.com/api/v1`; scores reset monthly on the service schedule.
- Keep the `HIGHSCORE_ID` and `MAX_SCORE` values synchronized between `index.html` and `highscores.html`.
- The high-score table ID is client-visible configuration. Never add the account ID, API tokens, or other credentials to source files. Dispatch credentials belong in the `SITE_DISPATCH_TOKEN` Actions secret.
- If API calls fail, the game falls back to browser `localStorage` under `scores_ns`.

## Local Verification

- Open `index.html` directly in a browser; no server is required for the game itself. Internet access is needed for the global leaderboard API.
- Test both SPACE and canvas taps. Check false starts, a sub-100 ms rejection, the under-500 ms deadline rule, the 500 ms halving boundary, game-over best time, and score submission/display conversion.
- There is no automated test command. Use browser-based checks for behavior and inspect the browser console for errors.

## Publishing

- The canonical game source is this repo's `main` branch.
- A push to `main` runs the trigger workflow, which sends a `repository_dispatch` event named `gofamreact-updated` to `charlesrg/gofamexplore.com` using `SITE_DISPATCH_TOKEN`.
- The site repo's Pages workflow checks out this repo into `/gofamreact` and deploys the combined static site. Confirm both Actions workflows succeed when diagnosing production updates. If dispatch is unavailable, the site repo workflow can be run manually.
- Do not copy built game files into the site repo; the deploy workflow checks out this repository at deploy time.