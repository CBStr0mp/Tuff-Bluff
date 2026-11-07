# Tuff Bluff

A bluffing word game for phones. Each round shows a real, strange word. Everyone secretly writes a fake definition, then everyone votes for the one they think is real. Picking the real definition scores 2 points. Each vote your fake gets scores 1 point.

3 to 15 players, 5, 8 or 10 rounds, 49 words. No accounts. Anyone with the link can play. With a big group the vote screen gets long (up to 16 answers, lettered A to P), so players scroll to find their pick.

## Put it online (about 10 minutes, free)

You need a free GitHub account and a free Render account. Do this on a laptop if you can. It is much easier than on a phone.

1. **Make a GitHub repository.** On github.com choose New repository, name it `tuff-bluff`, and leave it empty.
2. **Upload these files.** On the new repository's page choose "uploading an existing file", drag in everything from this folder (`server.js`, `package.json`, `render.yaml`, `public`, `test`, `README.md`, `.gitignore`), and commit.
   **Check the upload.** Open your repository on GitHub. You should see `server.js` and `package.json` at the top, and a folder named `public`. Click into `public` and make sure it contains `index.html`. Dragging a folder onto GitHub sometimes drops it. If `public` is missing, choose Add file, then Create new file, type `public/index.html` as the name (the slash makes the folder), paste in the contents of `public/index.html` from this zip, and commit.
3. **Create the service on Render.** At render.com choose New, then Blueprint (or New, then Web Service), connect your GitHub account and pick the repository. With a Blueprint, Render reads `render.yaml` and fills everything in. For a manual Web Service use:
   - Runtime: Node
   - Build command: `npm install --omit=dev`
   - Start command: `npm start`
   - Instance type: Free
4. **Wait for the first deploy.** When it says Live, Render shows your address, something like `https://tuff-bluff.onrender.com`.
5. **Send that address to your friends.** Open it yourself, enter a nickname and choose "Start a new game". Give everyone the 4-letter code.

## What to expect on the free plan

Checked against Render's free-plan page when this was built. Render can change its terms, so look at render.com/docs/free if something differs.

- **Cold start.** A free service goes to sleep after 15 minutes with no traffic and takes about a minute to wake. **Open the address a couple of minutes before game night** so it is awake. Messages during a game keep it awake. The page shows a "waking the server" note if someone arrives while it is asleep.
- **750 free hours a month.** That is plenty for a game night. A sleeping service does not use hours.
- **No saved data.** Games live in memory only. If the service restarts, games in progress end. That is fine for a party game, and nothing about your friends is stored.

Other hosts that run a Node server also work (Railway, Fly.io, a small VPS). The only requirement is that WebSockets are allowed, and the server listens on the `PORT` that the host provides.

## If something looks wrong

- **The address shows "Not found" or says the game page is missing.** The server is running but `public/index.html` did not upload. Fix the upload as described in step 2, and Render redeploys on its own. (A loose `index.html` next to `server.js` also works.)
- **The address shows a blank page, or "The game could not start".** The game file on the server is probably incomplete. Open your address followed by `/healthz` (for example `https://tuff-bluff.onrender.com/healthz`). It should say `game page: 156696 bytes`. If the number is much smaller, the file was cut off when it was uploaded or pasted. Upload `public/index.html` from this zip again as a file instead of pasting its text.
- **The address loads but "Start a new game" never connects.** Wait a minute, since a sleeping free service takes about that long to wake, then reload.
- **A deploy fails.** Open the service's Logs tab on Render and read the last few lines. The most common cause is that `package.json` did not upload.

## Try it on your own computer first

```
npm install
npm start
```

Open http://localhost:3000. To try it with phones on the same Wi-Fi, open `http://<your computer's local address>:3000` on each phone.

## Tests

```
npx playwright install chromium
npm test
node test/resilience.js
```

`npm test` plays a full 5-round game with three simulated phones and checks the words, voting, scoring and final screen. `test/resilience.js` checks a phone refreshing mid-round and the host dropping out. `test/crowd.js` plays a full game with 15 phones and checks that a 16th is turned away. `test/topics.js` checks the topic picker. `test/social.js` checks photos and chat. `test/rooms.js` checks that two games at once stay separate. `test/features.js` checks the skip button, paste blocking, the stepped-away note, reactions, marking a correct answer, and word repeat avoidance.

## Game features

- **Four topics, more than 800 entries.** Weird Words (340), Historical Events (167 obscure ones), Dates (153) and People (164 little known but significant figures from history). The historical entries were fact-checked against sources such as Wikipedia and Britannica. The host turns topics on or off in the lobby, and at least one must stay on. A date round shows a date and players write what happened that day. An event round shows an event name and players write what it was. A people round shows a name and players write who that person was and what they did. Definitions are plain (no semicolons or parentheses). Each phone remembers which words it has seen, and the host's game picks the least-seen words, so repeat games stay fresh.
- **Skip (host only).** The host can swap out a word during the writing stage. Tap twice to confirm. Everyone is told and rewrites their answer.
- **Emoji reactions** on the reveal screen, with the names of who reacted listed under each answer. They never change the score.
- **Mark as correct (host only).** If a bluff happens to be the real definition, the host taps Mark on that answer. The author gets 3 points, and anyone who voted for it gets the 2 points for spotting the truth. Tap Undo to reverse it.
- **No pasting** into the answer box, so nobody can look the word up and paste the definition.
- **Stepped away note.** If someone leaves the page for 5 seconds or more while writing, the reveal screen shows how long under their answer. It is not shown while voting, since that would give away whose answer it is.
- **Green real definition.** The real answer is shown in a green box on the reveal.
- **Photo avatars.** Players can choose a photo on the start screen or in the lobby. The phone crops it to a small square and compresses it before sharing, and it replaces the colored initial everywhere. A phone remembers its photo for next time.
- **Chat.** A Chat button sits in the corner during the whole game. It shows an unread count, keeps each person's last 8 messages (160 characters each), and clears when the game ends or a player leaves. With the chat closed, a new message pops up briefly above the button (tap it to open the chat). Sending a message closes the keyboard, and tapping the box brings it back. While the keyboard is up, the chat shrinks to fit the space above it.
- **Rooms stay separate.** The server only sends each phone the traffic from its own room, so several games can run at once without slowing each other.

## How it works

`server.js` serves the page and relays each phone's small state to every other phone. The game rules run in the browsers. The server only checks sizes and never reads game content.

- A phone that refreshes or loses signal rejoins on its own with its answer and vote kept. The server holds a dropped phone's state for 25 seconds.
- If the host's phone drops for good, everyone sees "The host dropped out" and any player can tap "Take over as host" to carry on from where the game stopped.
- The host, not the server, decides when a round ends: when every player has answered or voted, or when the host taps "Move on without the rest".

## Adding words

The word list is the `WORDS` array near the top of the script in `public/index.html`. Each entry is `{w:"word", p:"noun", d:"Its real definition."}`. Use real, obscure words with accurate definitions, since the game depends on the real one being true. Always add new words at the end of the list, and update the topic ranges in the `TOPICS` line just under it (`from` and `to` are positions in the list). Phones track seen words by position, so reordering or deleting entries makes the memory point at the wrong words.
