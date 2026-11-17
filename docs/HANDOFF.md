# Bronson's Big Day: project handoff

This file holds everything a new Claude account (or any developer) needs to keep working on the game. It was written on 2026-11-17, when the project moved off the Claude account it was built in. Before then, the project's notes lived in that account's project memory, and the playable copy lived in a Claude artifact. Both are gone with the old account, so this repo is now the only source of truth.

Read this together with `CLAUDE.md` (rules for Claude), `README.md` (player-facing features) and `leaderboard/README.md` (API and anti-spam).

## 1. What the game is

"Bronson's Big Day" is a 3D browser game about Brian's dog Bronson ("Bronny") and his two owners, Mom and Andy.

- Play: https://wickedhardflip.github.io/bronsons-big-day/ (GitHub Pages from `main`, folder `/`).
- Trailer: https://youtu.be/CsapECnSUrU (linked in the README).
- Repo: https://github.com/wickedhardflip/bronsons-big-day (public).
- Tech: Three.js r128 (cdnjs, with `vendor/three.min.js` as an offline fallback) and the Web Audio API. Every character, texture and prop is generated in code. The only audio files are four CC0 bark clips.
- Modes: a 6-chapter story, the Lake Haigler Dash race, Free roam (unlocked after a full run), the Wardrobe and Photo mode.

## 2. Characters (Brian's descriptions)

Bronson / Bronny
- About 80 lb chocolate / red-brown mix with a lean lab/pit build, floppy ears and amber eyes.
- Yellow collar with a black bone tag, blue harness.
- Real quirks that are in the game: he stretches often, he scratches his sides, and he stares into your soul. Leave him idle and these play (`Models.startQuirk`).
- He likes sleeping, treats, his dog friends and walks.

Mom
- Rule: she is only ever "Mom". Never use her real name anywhere (game text, code, comments, commits, docs, memory).
- Long light-brown hair in two braids, glasses, arm tattoos, white dress with blue florals.
- She loves dressing Bronson in dog clothes, which is why the Wardrobe exists.

Andy
- Red hair, blue shirt, jeans, brown boots (a camo cap in duck-hunt and fishing scenes).
- In Free roam he sits fishing at the end of the dock. You can talk to him, have him call Bronny over for a scratch, or have him toss Bronny a fish, which Bronny catches and eats.

Friend dogs (story Bark Park and the race's starter rivals), per Brian on 2026-10-22
- Dexter: brown mini schnauzer.
- Morgan: short 50 lb bulldog.
- Cookie: 12 lb black-and-white terrier mix.
- They replaced the invented Daisy, Moose and Pepper. Do not bring those names back.

## 3. Brian's working preferences

- Brian is an IT infrastructure engineer and values accuracy over speed. He is often on his iPhone.
- Replies in chat: plain text, no markdown, no code blocks unless it is actual code.
- End answers with an accuracy rating, and flag any assumption.
- Say so right away when unsure. Do not guess or hallucinate.
- Before a multi-step guide, ask whether he has given all the information, then ask whether he wants the full set or 2 to 3 steps at a time.
- When there are several approaches, list them briefly and let him choose.
- Group diagnostic questions together instead of asking one at a time.
- Professional, plain prose. Avoid em dashes and filler words.
- Decision cards or option widgets in Claude threads were sometimes not visible to him on iPhone. Restate any choice in plain text in the main chat too.
- Ship in increments: after each round, publish, push and post a short summary.

## 4. Hard rules

1. Never commit API keys, tokens, passwords or other secrets to any repo. This repo is public. Keep `.gitignore` covering env and secret files, and check the diff before every push.
2. Never use Mom's real name anywhere. She is "Mom".
3. Do not put Brian's email address or other personal details in the repo.
4. The D1 database `hippo-heist-leaderboard` also holds Brian's other game, Hippo Heist: tables `runs` and `scores` (plus `_cf_KV` and `sqlite_sequence`) and the Worker `hippo-heist-leaderboard`. Never drop, alter, write to or redeploy any of those. Bronson only uses tables named `bronson_*` and the Worker `bronson-leaderboard`.
5. Ideas in the backlog (section 9) are recorded only. Do not build them until Brian asks.

## 5. Code layout and build

- `src/shell.html`: page markup and CSS.
- `src/audio.js`: procedural music and sound effects (the barks are inlined by the build).
- `src/models.js`: Bronson, Mom, Andy, friend dogs, critters, props and the idle quirks.
- `src/outfits.js`: the Wardrobe (`Outfits.dress`), every look and its unlock rule.
- `src/world.js`: sky, terrain, grass, trees, house, lake, dock, duck blind, meadow, time of day.
- `src/course.js`: Lake Haigler trail geometry, 11 flag gates, cove, bridges, dam, hills. It also runs in Node with a global `THREE`.
- `src/race.js`: race path format (10 Hz frames, base64 int16), encode, decode, sample, progress, synth. Designed to be reused for live multiplayer.
- `src/game.js`: camera, controls, chapters, HUD, leaderboard client, race mode, Free roam.
- `vendor/`: three.js r128 and adapted Sky/Water shaders (MIT).
- `assets/sounds/`: four CC0 bark clips. Credits in `assets/CREDITS.md`.
- `leaderboard/`: the Cloudflare Worker source and D1 schema.

Build: `node build.js` writes `index.html` (the GitHub Pages page, with the offline three.js fallback) and `dist/bronson.html` (the body used for a Claude artifact; `dist/` is gitignored). Always commit the rebuilt `index.html` with any `src/` change.

Saved browser data uses localStorage keys prefixed `bronson.`: `balls`, `best`, `diff`, `finished`, `finishedHard`, `lbInitials`, `lbMine`, `outfit`, `pb`, `raceMine`, `racePB`, `roamOutfit`, `roamTod`.

## 6. Deploying

GitHub Pages
- Pages serves branch `main`, folder `/ (root)`. A push to `main` deploys within a minute or two.
- Prior rounds were pushed straight to `main`. The Pages settings API was blocked for Claude by its network proxy, so any Pages setting change is a manual step for Brian (on iPhone: request the desktop site in Safari, then Settings > Pages).

Claude artifact (optional)
- The old artifact (claude.ai/artifact/JXeQJuW749JfkgiSDuue2B) belongs to the closed account and is gone. On the new account, publish `dist/bronson.html` as a new artifact once, then always republish to that same URL so no duplicates appear. It needs the `downloads` capability for the photo Save button.
- The artifact cannot post leaderboard scores: the Worker only accepts POSTs from the GitHub Pages origin. Reading the boards works.

## 7. Leaderboard backend (Cloudflare)

- Worker: `bronson-leaderboard`, at https://bronson-leaderboard.bri-wag.workers.dev (the URL is hard-coded as `LB_URL` in `src/game.js`). Source: `leaderboard/src/index.js`.
- D1 database: `hippo-heist-leaderboard` (shared with Hippo Heist; see rule 4).
- Story tables: `bronson_runs`, `bronson_scores` (column `diff`, index `bronson_scores_diff`).
- Race tables: `bronson_race_runs` (id, started_at, ip_hash, used) and `bronson_race_times` (initials primary key, time_ms, path base64, created_at, ip_hash, ver).
- Schema: `leaderboard/schema.sql`, then `migrations/0002_difficulty.sql` and `migrations/0003_race.sql` for a database made before those features. The live database already has all of them.
- Story board: fastest full-game time, 3-letter initials, top 3 on the title screen and a Top 10 view with tabs All / Easy / Normal / Hard. All difficulties count; the All tab labels each row's difficulty (Brian, 2026-10-21).
- Race board: one row per set of initials, holding that player's best time and its path. Validation details are in `leaderboard/README.md`. The 11 flag coordinates are hard-coded in the Worker from `src/course.js`, so regenerate them if the course changes.

Bindings on the Worker
- `DB`: D1 binding to `hippo-heist-leaderboard`.
- `ALLOWED_ORIGINS`: plain text, `https://wickedhardflip.github.io`.
- `IP_SALT`: secret. It lives only in Cloudflare. Never commit it.

Redeploying the Worker (only `bronson-leaderboard`)
- Easiest: Wrangler with a local, uncommitted `wrangler.toml` (it is gitignored). Example, with the database id taken from the Cloudflare dashboard:

```
name = "bronson-leaderboard"
main = "leaderboard/src/index.js"
compatibility_date = "2025-09-01"

[[d1_databases]]
binding = "DB"
database_name = "hippo-heist-leaderboard"
database_id = "<from the Cloudflare dashboard>"

[vars]
ALLOWED_ORIGINS = "https://wickedhardflip.github.io"
```

  Then `npx wrangler deploy`. The existing `IP_SALT` secret stays in place. Never run Wrangler commands against the Hippo Heist Worker or tables.
- What earlier sessions did instead: a multipart PUT to the Workers scripts API (`/accounts/<account id>/workers/scripts/bronson-leaderboard`) with metadata `main_module: index.js`, `compatibility_date: 2025-09-01`, the `DB` and `ALLOWED_ORIGINS` bindings, and `keep_bindings: ["secret_text"]` so `IP_SALT` survives.
- After any live test, delete only the test rows you created, and only from `bronson_*` tables.

What the new account needs
- A Cloudflare API token with Workers Scripts edit and D1 edit rights on Brian's Cloudflare account, given to the session as the environment variable `CLOUDFLARE_API_TOKEN` (and the account id as an environment variable too). Never paste the token into the repo or chat logs that get committed.
- The Cloudflare connector, if Brian wants Claude to read Cloudflare directly.
- GitHub access to `wickedhardflip/bronsons-big-day` (connect GitHub and install the Claude GitHub App on the repo).

## 8. Ship log

All dates are 2026.

- Oct 21, first versions: 4 chapters, then v3 with crisper graphics, the real bark, Duck Hunt with Andy and Picnic with Mom. Dad renamed to Andy. Repo made public and Pages turned on by Brian.
- Oct 21, 15bd0b9: global leaderboard on Brian's existing D1 database, 3-letter initials.
- Round 1, 36af771: chapter goal cards that stay until tapped, landing rings, edge arrow, minimap, loading bar.
- Round 2, 3ae49c3: Bronson's quirks (stretch, side scratch, soul stare), people walk rewrite, arm IK and two-handed gun hold, 3D reed blind.
- Round 3, 8166526: difficulty Easy/Normal/Hard (Easy about 30% more time, Hard about 20% less; picnic clock fixed at 75 s), leaderboard tabs, run clock, personal best per difficulty, chapter select after a finished run (practice mode).
- Round 4, d763674 and 9ba97dd: a hidden tennis ball per chapter, the Wardrobe (28 looks), photo mode in the pause menu and wardrobe. Finishing unlocks holiday looks; finishing on Hard unlocks Halloween looks. 9ba97dd made photo Save work in the Claude viewer.
- Round 5, 4adc48e: Free roam. Unlocked by finishing; tapping the title logo three times quickly opens it even before. Mom in first person, 12 commands plus fetch, treat and pet, Andy fishing at the dock end. Free roam keeps its own outfit (`roamOutfit`, every look unlocked) and time of day (`roamTod`).
- Fix ee7baae: Brian could not find pause or quit on iPhone. Added a large "Menu" button top right with a collar-yellow border; pause menu says "Quit to main menu"; on short landscape screens the minimap moves beside the score.
- Race, 8198c6b (Oct 22): Lake Haigler Dash race mode with its own race board, and Dexter, Morgan and Cookie as story friends and starter rivals.
- b372428 (Oct 27): README links the trailer.
- Nov 17: this handoff (CLAUDE.md, docs/HANDOFF.md, docs/reference-photos/ if Brian approves).

Brian's picks from the Oct 21 idea list: 2 unlockable looks, 4 leaderboard extras, 5 difficulty, 6 photo mode, 7 polish. He did NOT pick 1 (real voices for Mom and Andy) or 3 (a fishing chapter).

## 9. Lake Haigler Dash decisions (Brian, 2026-10-22)

- A separate mode on the title screen, not inside the story. Open from the start, no difficulty levels.
- Course: one lap of the trail around Lake Haigler at Anne Springs Close Greenway, Fort Mill, SC. Landmarks: Nature Center / Gateway & Canteen start, campground on the west shore, two wooden bridges over lake arms, south shore, the dam with its spillway waterfall and the kayak center on the east. Dirt trail, dense forest, a few small hills. A lap takes 1 to 2 minutes (autopilot about 77 s, a casual first run 90 to 100 s).
- Obstacles: logs to jump, mud that slows, geese and squirrels crossing, hikers with leashed dogs, a horse and rider, a deer that startles and bounds off, blue zoomies treats for a speed boost. The cove swim shortcut is allowed but must not be clearly the best line.
- Rivals: up to 3. Either a 1 on 1 against a player picked from the board, or 3 random players. Rival Bronnys are about 50% see-through, each with a different collar color and initials floating above; the player's Bronny is the only solid dog. Do not call them "ghosts" in game text without asking Brian.
- HUD: live time, top times, "behind by".
- The race board is separate and shows only each player's best time; one set of initials is one player.
- Starter rivals while the board is thin: Cookie 87.6 s, Dexter 97.4 s, Morgan 111.8 s (pre-recorded, `STARTERS` in `src/game.js`).
- Build notes: race props and the race chapter object live in the "LAKE HAIGLER DASH" block of `src/game.js` (RC table of obstacle lap fractions). Dog breed looks are in `DOG_LOOKS` (options to `makeDog`).

## 10. Future work and backlog (do NOT build until Brian asks)

- Live multiplayer, later: Cloudflare Durable Objects with WebSockets, one object per room, starting with Free roam. Reuse the frame format in `src/race.js`. Leaning toward a separate Worker (for example `bronson-multiplayer`) so the leaderboard stays isolated; Brian has not chosen yet. On 2026-10-27 Claude said SQLite-backed Durable Objects probably run on the Workers Free plan with daily limits, otherwise the $5/month Workers Paid plan. That was not verified against Cloudflare's current pricing; check before relying on it.
- Deer Chase (idea recorded 2026-10-23): a multiplayer mode. One player is the deer, one is Bronny. A run through the woods with obstacles; Bronny chases and must catch the deer. Include hide and seek: the deer can hide and Bronny has to find it.
- App stores (discussed 2026-10-22, Brian said "just thinking for the future"): wrap with Capacitor for iOS and Android (Apple $99/yr, Google Play $25 once), or a free installable PWA first. A native rebuild in Godot was floated as a possible low-cost experiment on his Mac via Claude Code. Nothing started.

## 11. Testing notes

- Headless Chromium with SwiftShader works. `window.__bronson.sim(seconds)` advances game time; `__bronson.keys.KeyW` and similar move the dog.
- Real requestAnimationFrame frames still tick during screenshots, so wrap `requestAnimationFrame` to pass a frozen timestamp for exact shots.
- From `file://` the Worker refuses POST /run with 403 (origin not allowed). That is expected in local tests.
- A local Worker test harness (Node with `node:sqlite`) was used in earlier sessions but lived only in a scratch folder and is not in the repo.

## 12. Things that did not move with the repo

- The Claude artifact and its URL (see section 6 to publish a new one).
- The Claude project memory and chat history. Everything worth keeping from them is in this file.
- Secrets: the Cloudflare API token and the Worker's `IP_SALT`. The salt stays in Cloudflare and keeps working. The new account needs a token as described in section 7.
- Brian's original reference photos of Bronson, Mom and Andy, unless they were added to `docs/reference-photos/`.
