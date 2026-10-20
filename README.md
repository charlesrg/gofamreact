# gofamreact
Game to check your Reaction time

Open `index.html` in a browser (or enable GitHub Pages on the repo: Settings > Pages > deploy from branch root) and press SPACE.

Round 1 measures your reaction time to the monster behind the door. Each following round you must shoot within half the previous limit, or the monster jump-scares you.

## Online high scores
Scores are stored locally by default. For a single global list, create a free Firebase Realtime Database (rules allowing read/write on `scores`) and set `SCORES_URL` at the top of the script in `index.html`.
