# CLAUDE.md

Bronson's Big Day is a 3D browser game (Three.js r128 plus Web Audio) about Brian's dog Bronson ("Bronny") and his owners, Mom and Andy. Live at https://wickedhardflip.github.io/bronsons-big-day/.

Read `docs/HANDOFF.md` first. It holds the characters, Brian's decisions, the ship log, the leaderboard backend, deploy steps and the idea backlog. This repo is the only source of truth; there is no other project memory.

## Rules (from Brian, always apply)

- This repo is public. Never commit API keys, tokens, passwords, the Worker's IP_SALT, or Brian's email. Check the diff before every push.
- Mom is only ever "Mom". Never write her real name anywhere.
- The D1 database `hippo-heist-leaderboard` also holds Hippo Heist (tables `runs`, `scores`, Worker `hippo-heist-leaderboard`). Never touch them. Bronson uses only `bronson_*` tables and the `bronson-leaderboard` Worker.
- Backlog ideas (Deer Chase, live multiplayer, app stores) are recorded only. Do not build them until Brian asks.
- Friend dogs are Dexter (mini schnauzer), Morgan (bulldog) and Cookie (small terrier mix).

## Working with Brian

- Replies: plain text, no markdown, include an accuracy rating, flag assumptions, say so when unsure.
- He is often on an iPhone. Restate any choice in plain text; option cards may not show for him.
- Before a multi-step guide, confirm he has given everything, then ask: full set or 2 to 3 steps at a time.
- Ship in increments and post a short summary after each.

## Build and ship

- Edit `src/`, then run `node build.js`. It writes `index.html` (commit it) and `dist/bronson.html` (gitignored; for a Claude artifact).
- `main` deploys to GitHub Pages from the repo root.
- Worker source is `leaderboard/src/index.js`; schema and migrations are beside it. See `docs/HANDOFF.md` section 7 before redeploying.
