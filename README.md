# Desk Pal kit

Set up your own copy of a small personal ecosystem: sticky notes on your Mac and phone, an assistant in your
Ray-Ban Meta glasses, and a little desk companion with a screen. All of it runs on your own accounts. Nothing
points at anyone else's server.

**Using a coding agent?** Open this folder in it and say:

> Set me up by following AGENTS.md.

The agent does the setup and asks you whenever it needs a login, a click in a web dashboard, or the hardware.

## What you get

| Piece | What it does | Repo |
|---|---|---|
| **Noty** (Mac) | Sticky notes that sit at the bottom edge of your screen, with checkbox tasks | [evxnche/noty](https://github.com/evxnche/noty) |
| **Noty** (phone) | A web page you add to your Home Screen. It syncs notes and tasks with the Mac | same repo, `web/` folder |
| **Desk Pal for glasses** | An iPhone app. Say "Gigi" to your Ray-Ban Metas and ask anything. It can take notes and tasks into Noty, read saved articles aloud, nudge your posture, and switch Mac Spaces when you turn your head | [evxnche/ezmeta](https://github.com/evxnche/ezmeta) |
| **Desk Pal server** | The brain behind the glasses and the desk board: speech to text, the AI, text to speech | [evxnche/desk-voice](https://github.com/evxnche/desk-voice) |
| **Desk Pal board** | A pixel crab on a 4.2" reflective screen. Voice assistant, Noty tasks, room temperature, flight radar | same repo, `board/` folder |
| **Reader inbox** (optional) | Saves articles so the glasses can read them aloud | [evxnche/reader-inbox](https://github.com/evxnche/reader-inbox) |

```
   Noty Mac app ─┐                                  ┌─ Ray-Ban Meta glasses ← iPhone app (ezmeta)
                 ├─ Noty server (noty/web) ─────────┤
   Noty phone ───┘   notes + tasks                  └─ desk-voice server ─── Desk Pal board (ESP32)
                                                        voice + AI
```

## What you need

| | Needed for |
|---|---|
| A Mac with Xcode | everything |
| An iPhone (iOS 17.2 or newer) | glasses app, Noty phone page |
| Ray-Ban Meta glasses, paired in your Meta AI app | glasses app |
| Waveshare ESP32-S3-RLCD-4.2 board ([Amazon UK](https://www.amazon.co.uk/dp/B0GF2616RN)) + a USB-C data cable | desk board |
| Free accounts: GitHub, Vercel, Apple ID, Meta Wearables Developer Center | everything |
| A few dollars of Vercel AI Gateway credit (the free monthly credit covers light use) | voice answers |

You can stop after any piece. Noty on its own works fine without the glasses or the board.

## Time

- About 1 hour: servers, Noty, glasses app.
- Plus about 1 hour: the desk board.

A free Apple ID signs the iPhone app for **7 days**. After that it stops opening and you rebuild it (one
command; AGENTS.md covers it). A paid Apple Developer account ($99/year) makes it last a year.

## Getting updates

Each piece lives in its own repo. This kit only holds the instructions. To get new features later, pull
from upstream in each repo (AGENTS.md → "Updating").
