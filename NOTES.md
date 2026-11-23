# NOTES - setting up Claude Code on this project

## What I put in CLAUDE.md, and what I left out

I kept the four parts the project needs and nothing else. The first line says what the API does. **Commands** lists `dev`, `test` and `lint`. It also records that lint must pass, because CI runs lint before the tests. **Conventions** holds five rules Claude can follow without asking:
- CommonJS only.
- One router per resource, mounted only in `server.js`.
- Data access only through `db/store.js`.
- JSON `{ error }` responses with 400 or 404.
- A supertest test for every route, with no real port opened.

**Architecture** is four lines covering `server.js`, `routes/`, `db/store.js` and where config comes from.

I deliberately left out:
- **The file list and the Express/Node basics.** Claude sees those the moment it opens the folder.
- **The seed users.** They are data, not rules.
- **Anything from `.env`.** No values and no example secrets, because CLAUDE.md is committed and loads into every session.
- **One-off tasks.**

Claude drafted it from reading the code, and I kept it lean on purpose. What remains is the rules that are not visible from reading a single file, such as "never touch `users` directly" and "never start a port in tests".

## Permission rules (`.claude/settings.json`, committed)

- **allow:** `npm test`, `npm run lint`, `git status`, `git diff`. These are safe, read-only or check-only commands I run constantly, so approving each one is just noise.
- **ask:** `git push` and `npm install`. A push reaches the shared remote, and an install changes dependencies and the lockfile. I want to see both before they happen.
- **deny:** reading `./.env` and `./.env.local`, force-push (`--force` / `-f`) and `rm -rf`.

Without the deny rule, Claude could open `.env` while "looking at the config". The secret would then sit in its context and could end up in a summary, a commit message or a pasted log. A force-push can silently overwrite teammates' work on the remote. A CLAUDE.md line saying "never read .env" is only guidance, which is why the deny rule is the real lock.

`.claude/settings.local.json` stays personal and is git-ignored. See the finding below.

## Finding while setting up: `.env` was not actually git-ignored

The starter `.gitignore` had every line after the first indented with spaces. Git does not trim leading spaces from a pattern, so `   .env` never matched `.env`. I checked this: with a test `.env` file present, `git status` listed it as untracked and ready to commit, and `.claude/settings.local.json` had the same problem. I removed the indentation in a separate commit. The deny rule and the ignore rule now protect `.env` from two sides.

## Verification

- **Install and sign-in.** Claude Code is installed (`claude` is on the PATH at `/opt/node22/bin/claude`). I did this work inside a Claude Code session in a cloud workspace.
- **Loading.** I registered the project folder with the session, and the new `CLAUDE.md` loaded straight into Claude's context as project memory, which is the same list `/memory` shows. Asked "how do I run the tests here?", it answers `npm test` from the file. I did not run `/memory` itself.
- **Permissions.** `.claude/settings.json` is valid JSON with the allow / ask / deny lists above. I could not run `/permissions` in this headless cloud session, because the CLI there cannot open an interactive login. On a local machine, the next step is to run `/permissions` and check that the three lists appear.
- **Project health.** `npm test` passes all 4 tests and `npm run lint` is clean, with no app code changed.
