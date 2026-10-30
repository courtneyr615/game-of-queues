# Game of Queues: simultaneous multiplayer

This package is a separate Node.js app. It does not change your Game of Life / ASNF server or the saved single-device Queues preview.

## Put it on Render

1. Create a new GitHub repository for Game of Queues.
2. Upload the contents of this package folder into the repository root. `package.json`, `server.mjs`, `engine.mjs`, and `public/` should be at the root, not inside another folder.
3. In Render, create a new Web Service using that repository. You can use the included `render.yaml` blueprint. The supplied blueprint requests the Starter plan with a persistent disk; review Render's plan details before creating the service.
4. Build command: `npm test`. Start command: `npm start`. Node version: 22. Health check: `/healthz`.
5. For saved games, mount a persistent disk at `/var/data` and set `DATA_FILE` to `/var/data/queues-rooms.json`. Use one running instance: the first-solver race is coordinated by this server. Without the persistent disk, rooms may be lost when the server restarts or redeploys.
6. Open the new Render address. One person creates a room, everyone else joins with that room code on their own device, and the host selects Start game.

Start a new room for this version. Old turn-based room files cannot be resumed by this package.

## Everyone can investigate at once

- No turn order and no End turn button. Each player's movement, dice, pop-ups, clues, responses, and notebook are independent.
- Each player starts with 20 points.
- Each player may solve each room once for 50 points, even if another player solved it earlier.
- The first correct accusation received by this server in a room gets an additional 25 points, exactly once. Every later first solve by another player gets 50 points.
- A repeated request or repeated solve never earns points again.
- Each roll gives enough movement to reach at least one other personally unsolved room where one is available. Normal dice faces remain 1-6; routes requiring more than 12 show an extra travel bonus.
- Entering a room ends that move. Players can roll again immediately. New move discards unused movement and allows another roll.
- Secret passages name their destination and cost 10 points per use.
- A room clue costs 5 points per player, once. It automatically saves to that player's room notes. Reopening it does not charge again.
- Solved rooms show a red SOLVED badge in that player's notes and checklist. Another player's solve does not mark your room solved.
- After your correct accusation, you can type an answer to "Can you identify the patient murdered in the [room name] Room?" or skip. A configured correct answer earns 50 points once; skipping or answering incorrectly costs nothing.
- Without a configured patient answer, the response is saved privately as pending review, with no points automatically awarded. There is currently no facilitator scoring screen.
- The game ends after every remaining player solves all eight mystery rooms and answers or skips their last bonus. Answers pending review are not included as bonus points in automatic final totals.
- Leaving removes that player from the completion requirement. Closing a tab alone keeps the session available to reconnect.

## Room scenarios and patient answers

Casework, Data Entry, Identification, Supervisors, Prescription Resolution, Orders, and Communication Error have supplied scenarios and fixed accusations. Data Entry uses Special Agent One / Computer Mouse with Kelsey Hall as the bonus answer. Identification uses Judge Peacock / Paper Cutter with James Ash as the bonus answer. Supervisors uses Professor Plum / Magnifying Glass with John Street as the bonus answer. Prescription Resolution uses Doctor Scarlet / Poison and accepts Gustav Garcia or Gus Garcia for the patient-identification bonus. Orders uses Doctor Scarlet / Ice and accepts Lennie Lesch for the patient-identification bonus. Communication Error uses Professor Plum / Telephone Cord and accepts Kathy Dailey for the patient-identification bonus. Desktop is a thank-you room awarding 25 points once per player. No accusation, paid clue, or patient bonus is needed there. AE/PC/OSF uses Detective Green / Poison and accepts Goldie Adams for the patient-identification bonus.

Edit `room-scenarios.json` on the server to add each room's person, weapon, challenge text, clue, and correct patient name (`patientName`). Optional alternative correct names go in a `patientAliases` array. Valid people and weapons are listed in `engine.mjs`. Both person and weapon must match those lists to use a fixed accusation; otherwise that room's accusation is randomized.

Keep this file outside `public/`. It contains the answer key. Changes apply to new rooms; existing rooms keep their accusation and patient answer from when they started. Patient names should be fictional training data.

## Run locally

With Node.js 22 installed, open a terminal in this folder and run:

```
npm test
npm start
```

Open `http://localhost:3000`. For a multiplayer test, use separate browser profiles or devices. A browser's stored device identity prevents it from claiming a second active seat in the same room.

## Validation

The build runs server-rule regressions and a four-client simulation. Local HTTP testing also verified authentication, simultaneous rolls, private state, static serving, and persistence after restarting the server.
