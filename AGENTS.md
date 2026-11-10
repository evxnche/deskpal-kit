# Instructions for the coding agent

You are setting up the Desk Pal ecosystem for the person in front of you, on their own accounts. README.md
says what the pieces are. This file is the order to do them in and how to check each one.

## Rules

1. **Ask the human for anything that needs them:** signing in (GitHub, Vercel, Apple ID, Meta), clicking
   in a web dashboard, accepting terms, plugging in hardware, and anything on the glasses or the iPhone
   screen. Tell them exactly what to click, then wait.
2. **Never commit a secret.** Keys, tokens, Wi-Fi passwords and team IDs go only in the gitignored files
   named below, in Vercel's environment variables, and in `setup.local.env` in this kit (gitignored).
   Before every commit in any repo, run `git status` and check that no secret file is staged.
3. **Each repo's README is the source of truth** for its own details. This file links to the right sections.
   If this file and a repo README disagree, follow the README and tell the human about the mismatch.
4. **Check each phase before starting the next one.** Every phase ends with a check. If it fails, fix it
   before moving on.
5. **Ask before skipping a phase.** Phases 5, 7 and 8 are optional. Ask the human which ones they want.

## Phase 0: tools and accounts

Ask which pieces they want: Noty only, plus glasses, plus the desk board.

Install what's missing:

```sh
xcode-select -p || echo "install Xcode from the App Store first"
brew install xcodegen gh node
npm i -g vercel
gh auth login       # the human signs in
vercel login        # the human signs in
```

For the desk board, also:

```sh
python3 -m pip install --user esptool mpremote
```

Create `setup.local.env` in this kit and record every host and key there as you go. Tell the human to
also save the keys in their password manager.

## Phase 1: fork and clone

Fork each repo the human wants into their own GitHub account and clone it into `repos/` in this kit
(gitignored). Forks let them deploy on Vercel from their own GitHub and change things without
affecting anyone else.

```sh
mkdir -p repos && cd repos
gh repo fork evxnche/noty --clone --default-branch-only
gh repo fork evxnche/desk-voice --clone --default-branch-only     # glasses or board
gh repo fork evxnche/ezmeta --clone --default-branch-only         # glasses
gh repo fork evxnche/reader-inbox --clone --default-branch-only   # optional: articles read aloud
```

`gh repo fork --clone` sets `upstream` to the original repo. "Updating" below uses it.

**Check:** `repos/noty/web/README.md` exists. If it doesn't, the sync code isn't on `main` yet: run
`git -C repos/noty fetch upstream && git -C repos/noty branch -r` and ask the human which branch to use.

## Phase 2: Noty server (notes and tasks in the cloud)

Follow `repos/noty/web/README.md` → **Deploy your own**, steps 1 to 5. In short: make `NOTY_SYNC_TOKEN`
with `openssl rand -hex 32`, `vercel link` in `web/`, add Upstash Redis in the Vercel dashboard (the human
clicks; free plan, eviction off), `vercel env add NOTY_SYNC_TOKEN production`, `vercel --prod`.

Record `NOTY_HOST` and `NOTY_SYNC_TOKEN` in `setup.local.env`.

**Check:** the `curl` in step 5 prints `200`.

## Phase 3: Noty Mac app and phone page

```sh
cd repos/noty
./build.sh release run
```

Then the human, on the Mac: right-click the Noty pill at the bottom of the screen → **Task Sync…** →
Sync site `https://<NOTY_HOST>` (root only, no path) → Private key = `NOTY_SYNC_TOKEN` → tick **Sync notes
and tasks** → **Save settings**. Then they scan the QR code under "Connect your phone" with the iPhone camera,
open the page, and Share → **Add to Home Screen**.

For the glasses' "add X to my tasks" and for the board's Plan screen, they also make a note called
**Desk Pal** and pick it as the Desk Pal note in Task Sync.

**Check:** a task typed on the phone page shows up in the Mac note within about 30 seconds, and the
other way round.

## Phase 4: desk-voice server (the voice brain)

Follow `repos/desk-voice/README.md` → **Deploy your own (glasses server)**, steps 1 to 7. Every variable is
explained in `repos/desk-voice/.env.example`. The human has to:

- import the fork in Vercel (vercel.com/new),
- add Upstash Redis to the project,
- turn on **AI Gateway** and add credit.

Set at least `VOICE_KEY` (`openssl rand -hex 24`), `NOTY_NOTES_URL` = `https://<NOTY_HOST>/api/notes`,
`NOTY_SYNC_TOKEN`, and `DESK_TZ` + `DESK_PLACE` for where they live. Both default to India. Then redeploy.

Record `VOICE_HOST` and `VOICE_KEY`.

**Check:** run the five `curl`/`say` lines under "Check it" in that README. The last one returns
`{"heard":"What time is it?","reply":"..."}`.

## Phase 5 (optional): reader inbox

Only if they want "read my articles" on the glasses. Follow `repos/reader-inbox/README.md` →
**Deploy your own**. Record `READER_HOST` and `READER_KEY`.

**Check:** the `curl` lines under "Check it" in that README.

## Phase 6: glasses app

Follow `repos/ezmeta/HANDOVER.md` steps 3 to 6. Skip its steps 1 and 2: phases 2 to 5 did them.

- `Config/Secrets.xcconfig` is gitignored. Fill it from `setup.local.env`. Host names go in without
  `https://`. Leave `READER_*` empty if phase 5 was skipped.
- `BUNDLE_ID` must be the human's own, e.g. `com.<theirname>.deskpal`. Use the same one in their Meta
  Wearables Developer Center project.
- The glasses must be paired to the human's own Meta AI app with **Developer Mode** on. Only one
  developer app can be registered with the glasses at a time.

**Check:** the human says "Gigi", hears the chime, asks "What's the weather?" and gets a spoken answer.
Then "Note: kit works", and that line appears in the Noty **Inbox** note.

Tell the human: with a free Apple ID the app stops opening after 7 days. Rebuild it with HANDOVER.md
step 5.3.

## Phase 7 (optional): desk board

The board is a Waveshare ESP32-S3-RLCD-4.2. It runs MicroPython; the app is the `board/` folder of
desk-voice. Read `repos/desk-voice/README.md` from the top to "Deploy your own" first.

1. **Flash MicroPython (v1.29) onto the board.** Use the ESP32-S3 build from micropython.org that matches
   the board's flash and PSRAM. Check the board's spec sheet for those; don't guess. The human plugs the board
   in with a data cable and may need to hold BOOT while pressing RESET. Skip the custom firmware in
   `firmware/`: it only adds the "gee gee" wake phrase and needs ESP-IDF. Without it, holding KEY talks to
   the assistant.
2. **Make `secrets.py`** in the desk-voice repo root from `board/secrets.example.py`: their 2.4 GHz Wi-Fi,
   `VOICE_URL` = `https://<VOICE_HOST>/api/voice`, `VOICE_KEY`, `READ_KEY = ""`. Add `secrets.py` to that repo's
   `.gitignore` first; it isn't listed there yet.
3. **Make `noty-sync.json`** (already gitignored) with `{"enabled": true, "url": "https://<NOTY_HOST>/api/sync",
   "token": "<NOTY_SYNC_TOKEN>"}`. See "Noty task sync" in that README.
4. **Set the room clock's time zone:** `TZ` in `board/room.py` supports `"India"` and `"UK"`.
5. **Upload** from the desk-voice repo root:

   ```sh
   python3 -m mpremote cp board/*.py board/*.bin board/noty-sync-ca.der : + cp secrets.py :secrets.py + cp noty-sync.json :noty-sync.json + reset
   ```

**Check:** the crab shows up, the board joins Wi-Fi, holding KEY and asking "what time is it?" gets a spoken
answer, and the Plan screen shows the tasks from the Desk Pal note.

Known gap: the usage-limits screen reads from a server only the original author runs
(`agent-limits.vercel.app` in `board/main.py`). With `READ_KEY = ""` that screen stays empty. Leave it.

## Phase 8 (optional): Mac extras

- **Head Spaces** (turn your head to switch Mac Spaces, tilt to scroll): `repos/ezmeta/HANDOVER.md` step 7.
- **Agent pager** (the glasses and board speak Claude Code / Codex alerts): `repos/desk-voice/README.md`, the
  "Agent alerts from your Mac" paragraph.
- **Mac control from the board** (lock, open apps, dictation): `repos/desk-voice/README.md`, the "Mac control"
  paragraph.

The launch agent files are named `com.evan.*` and some contain `/Users/evancheriyan` paths. Copy each one,
rename it to `com.<theirname>.*`, change the label inside to match, and replace the path with the human's
`$HOME`. Don't edit the originals in the repo.

## Make it theirs

Things that are set for the original author. Change them only if the human wants:

| What | Where |
|---|---|
| Wake word "Gigi" | ezmeta: search for `Gigi`. Board: `WAKE_PHRASES` in `board/audio.py` (custom firmware only) |
| Assistant voice | `TTS_VOICE` env var on desk-voice |
| Time zone and place | `DESK_TZ`, `DESK_PLACE` (desk-voice env), `TZ` in `board/room.py` |
| Calendar on the glasses and board | `PLAN_CALENDARS` env var: private iCal links, space-separated |

## Updating

When the original author ships something new, in each repo:

```sh
git fetch upstream && git merge upstream/main
```

Then redeploy the server (`git push` to the fork; Vercel deploys it), rebuild the app (Noty: `./build.sh
release run`; glasses: HANDOVER.md step 5.3) or re-upload the board files. Their local secret files aren't
touched by a merge.
