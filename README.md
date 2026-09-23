# Heisenbot

A Walter White reaction bot for your Messenger group chat. It watches the chat through a real Chromium window logged in as a second Facebook account. When someone says a trigger word or tags `@Walter`, it picks a matching clip, burns in the sender's name and message, adds a Walter voice line, and posts the video.

Two ways to run it:
- **Cloud (recommended, your PC can be off):** a small Linux server runs Walter 24/7 and GitHub Actions tests, deploys and watches it. You control him from your phone through a password-protected web page. See [Set and forget](#set-and-forget-one-time-setup-about-20-minutes).
- **Your own PC:** double-click `start_windows.bat`. The control room opens at `http://127.0.0.1:8765`.

## Right now: modes for real life

The dashboard's first card has one-tap modes. Each one switches itself off when its time runs out, so you can't forget to undo it.

| Button | What Walter does | How long |
|---|---|---|
| Class | Only answers when someone tags @Walter | 2 h |
| Meeting | Only answers tags | 90 min |
| Date | Completely quiet | 4 h |
| Errands | Completely quiet | 2 h |
| Sleep | Completely quiet | 10 h |
| Back to normal | Reacts as usual | now |

**Without opening the dashboard.** In the ntfy app, subscribe to a second topic: your alert topic plus `-cmd` (for example `walter-8fj2k1la0c-cmd`). Send it any of these:

| You send | Walter does |
|---|---|
| `class` | Only answers tags, 2 h |
| `meeting` | Only answers tags, 90 min |
| `date` | Quiet, 4 h |
| `out` | Quiet, 2 h |
| `pause 3h` | Quiet, 3 h |
| `mentions 45` | Only answers tags, 45 min |
| `resume` | Back to normal |
| `status` | Replies with what he's doing |

Walter checks for your message every 15 seconds and replies in the app.

**Care guard (on by default).** When someone sends something serious, Walter stays silent in the chat for 45 minutes, even if people tag him. Examples: a death, a hospital visit, a breakup, a panic attack, "ayoko na mabuhay". The word list covers English and Taglish and is editable under Behavior. The dashboard shows when the guard is on, with a button to let him talk again.

**Green screen clips.** If a clip's edges are mostly bright green, Walter swaps the green for a dark lab backdrop before sending. Behavior → Green screen clips: automatic, always, or never.

**Unknown names.** When Walter can't read who sent a message, he no longer writes "Someone" or shows a "So" tile. He just leaves the name out: "What are you talking about?"

## Run it 24/7 in the cloud

Walter needs a real Chromium session that stays logged in for weeks, so he lives on a small Linux server. GitHub Actions does the rest for you: it tests every push, deploys it to the server, and checks on him every 3 hours.

```
push to main ──► Actions: tests ──► Actions: SSH deploy ──► server (systemd, 24/7)
                                                              ▲
Actions: watchdog every 3 h ──────────────────────────────────┘
server itself: health check every 5 min, update check nightly, restart daily
```

### Why not run the bot inside GitHub Actions?
| | |
|---|---|
| Terms | GitHub's terms allow Actions "only to develop and test your application(s)". Using it as a host can get your account suspended. |
| Time limit | A job is killed after 6 hours, so there would be gaps. |
| Facebook | Every run gets a new machine and IP address. Facebook sees a new login from a new data centre every few hours and locks the burner account fast. |

So Actions does the testing, deploying and watching, and the server does the running.

### Where it can run
| Option | Cost | Notes |
|---|---|---|
| **Oracle Cloud Always Free**, Ampere A1 (Arm), 2 CPU / 12 GB | 0 | Sign-up needs a card for identity. You may see "out of capacity"; try another zone later. Idle free servers can be reclaimed, so switching to Pay As You Go (still 0 on free-tier shapes) is safer. |
| **Hetzner CX23**, 2 CPU / 4 GB | about EUR 4 to 6 a month | Easiest option. Choose Ubuntu 24.04. |
| Any Ubuntu 22.04 / 24.04 VPS, 2 GB RAM or more | varies | Works on x86 and Arm. |

## Set and forget: one-time setup (about 20 minutes)

### 0. Turn on the workflows (once)
The workflow files are in `deploy/workflows/`. GitHub only runs them from `.github/workflows/`. Easiest way: open the repo, press `.` to open github.dev, drag the three files from `deploy/workflows/` into a new folder `.github/workflows/`, then commit. Or from a terminal:
```bash
git clone https://github.com/tiredicey24-hub/Heisenbot && cd Heisenbot
mkdir -p .github && git mv deploy/workflows .github/workflows
git commit -m "Enable workflows" && git push
```

### 1. Get a server
Create an **Ubuntu 24.04** server with one of the options above. When it asks for an SSH key, paste the **public** key from step 2. Write down its **public IP address**.

### 2. Make a deploy key (on your PC)
Open PowerShell (Windows) or Terminal (Mac/Linux):
```bash
ssh-keygen -t ed25519 -f heisenbot_deploy -N ""
```
You get two files:
- `heisenbot_deploy.pub` is the **public** key. It goes on the server.
- `heisenbot_deploy` is the **private** key. It goes into GitHub. Never share it anywhere else.

If the server already exists, add the public key to it:
```bash
ssh ubuntu@YOUR_SERVER_IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" < heisenbot_deploy.pub
```
Test it with `ssh -i heisenbot_deploy ubuntu@YOUR_SERVER_IP "sudo -n true && echo ok"`. It must print `ok`. The login user needs passwordless sudo, which `ubuntu` on Oracle and `root` on Hetzner already have.

### 3. Add secrets to GitHub
Go to **github.com/tiredicey24-hub/Heisenbot → Settings → Secrets and variables → Actions → New repository secret**:

| Name | Value |
|---|---|
| `SERVER_HOST` | Server IP, e.g. `203.0.113.10` |
| `SERVER_USER` | `ubuntu` (Oracle) or `root` (Hetzner) |
| `SERVER_SSH_KEY` | Whole contents of `heisenbot_deploy`, including the `-----BEGIN` and `-----END` lines |
| `SERVER_PORT` | Optional, only if SSH isn't on port 22 |
| `NTFY_TOPIC` | Optional, fill in after step 5 so the watchdog can alert your phone |

### 4. Deploy
**Actions tab → Deploy → Run workflow**. The first install takes about 5 to 8 minutes. When it's green, Walter is running.

From now on, **every push to `main` is tested and deployed automatically**. If tests fail, nothing is deployed.

### 5. Get your dashboard link and password
Passwords never appear in the Actions logs. SSH in once:
```bash
ssh -i heisenbot_deploy ubuntu@YOUR_SERVER_IP
sudo walter
```
It prints the dashboard address (`https://....trycloudflare.com`), the password and your **ntfy topic**. Install the **ntfy** app on your phone and subscribe to that topic. The new dashboard address is sent there whenever it changes, along with login and crash alerts. Put the same topic in the `NTFY_TOPIC` secret too.

### 6. Log Walter into Facebook (from your phone)
1. Open the dashboard link and enter the password.
2. Tap **Open browser**. The **Remote browser** panel shows the server's Chromium.
3. Tap and type on the screenshot to log in as the **burner** account and pass any checks.
4. Open the group chat, tap **Save chat**, then **Go live**. Leave **Dry run** on until the log looks right.

Or log in on your PC with `start_windows.bat`, use **Export login**, then **Import login** on the server dashboard.

After the bot is live, it resumes by itself after crashes, restarts, updates and reboots. You're done.

### What keeps it alive
| Layer | What it does |
|---|---|
| systemd | Restarts the bot 10 s after any crash, and starts it on boot |
| Daily restart | Fresh Chromium every 24 h to stop memory creeping up |
| Health timer (5 min) | Restarts the bot if `/healthz` stops answering or the chat hasn't been read for 5 min |
| Nightly update (04:30) | Pulls `main` from GitHub, restarts only if something changed |
| Actions watchdog (3 h) | SSHes in, runs the health check, sends a phone alert if the server can't be reached |
| Housekeeping | Old renders deleted, logs capped at 200 MB, 2 GB swap on small servers, automatic security updates |

### Commands on the server
| Command | What it does |
|---|---|
| `sudo walter` | Dashboard address, password, ntfy topic |
| `sudo walter status` | Bot, tunnel and health state, commit, disk, memory |
| `sudo walter logs 200` | Last 200 log lines |
| `sudo walter restart` | Restart bot and tunnel |
| `sudo walter update` | Pull the newest code now |
| `sudo walter password mynewpass` | Change the dashboard password |
| `sudo walter stop` / `sudo walter start` | Pause / resume. While stopped, the health check and updates leave him off |

Manual install without Actions:
```bash
curl -fsSL https://raw.githubusercontent.com/tiredicey24-hub/Heisenbot/main/deploy/install.sh | sudo bash
```

### The only things you might ever do
| When | Do |
|---|---|
| ntfy says "Walter needs you" | Facebook wants a login or check. Open the dashboard → Remote browser and complete it |
| ntfy watchdog alert | Check the server is running in your cloud provider's console, then `sudo walter status` |
| You want to change code | Push to `main`. Actions tests and deploys it |
| Scheduled workflows stop | GitHub pauses schedules in repos with no activity for 60 days. Actions → Watchdog → Enable workflow. The server keeps running either way |

## Read this first: is it possible?

Yes, with limits. The facts, with sources:

| Claim | Status | Source |
|---|---|---|
| messenger.com stopped working for messaging in April 2026. Web chat now lives at `facebook.com/messages`. | Confirmed | TechCrunch, 19 Feb 2026, "Meta is shutting down Messenger's standalone website" |
| Meta's Terms forbid accessing its products "using automated means (without our prior permission)". A bot account breaks this rule, whatever tool drives the browser. | Confirmed | facebook.com/terms (Meta Terms of Service, "What you can share and do") |
| Browser automation "prevents API detection bans" | **False.** No tool can promise this. Using a real browser looks more like a normal user than an unofficial API does, but Meta can still flag and disable the account. | Terms above. Nobody can verify an "undetectable" claim. |
| VTube Studio can export Walter as a 3D model | **False.** VTube Studio only supports Live2D (2D) models. For 3D use VRoid, VSeeFace, or Blender. | VTube Studio official documentation: "only supports Live2D models. VRoid/VRM is not supported." |
| edge-tts is free, needs no API key, and works from Python | Confirmed, version 7.2.8 (22 Mar 2026). It calls Microsoft's online Edge voice service, so it needs internet. The service is unofficial and Microsoft could change it. | pypi.org/project/edge-tts |
| Encrypted Messenger chats need a PIN on each new device | Confirmed | Facebook Help Center, "Restore end-to-end encrypted chats with a PIN" |

What this means for you:
- Use a **burner account only**. Never your main one. It can get restricted.
- Keep the account's activity low and human-like. The defaults are 20 s cooldown, 45 s per person, and 25 clips an hour at most.
- Tell your group Walter is a bot. It's their chat.
- Don't use real Breaking Bad footage or the actor's real voice. Use your own model and the built-in synthetic voice. The Walter look and lines are fan parody. Keep it inside your private group.

## What it does

- **Clip bank.** One folder per reaction: `clips/laugh/`, `clips/facepalm/`, and so on. Drop in as many MP4s as you like. It won't send the same clip twice in a row.
- **Starter pack.** No animations yet? Upload one picture of your Walter model and it builds 8 animated 3-second clips (bounce, sink, sway, shake, zoom, tilt, fade, push). That's enough to test before you make real ones.
- **Smart triggers.** Keywords match whole words, so "fail" doesn't fire inside "failsafe". Case doesn't matter. Stretched spellings count: "lmaooooo", "bruhhhh" and "HAHAHAHA" all match. The longest match wins, so "wala magawa" beats "lol". Emoji keywords work (💀 😂 🤔). Taglish keywords come preloaded.
- **Mentions.** `@Walter` or `@bot` always gets a reply, even during that person's cooldown.
- **Captions.** A Breaking Bad style periodic-table tile with the sender's initials, their name and message across the top, and Walter's line in big meme text at the bottom.
- **Voice lines.** A deep edge-tts voice (Christopher, pitched down) reads Walter's line, mixed over the clip's own audio. The clip loops if the voice runs longer. You can have it read the message instead, or both.
- **Lurk mode.** Plain messages have a 3% chance (you can change this) of getting a silent stare.
- **Chat commands.**

  | Command | Effect |
  |---|---|
  | `!walter` | Help |
  | `!walter list` | Show reactions |
  | `!walter rage stop spamming` | Play that reaction with your own line |
  | `!walter off` / `!walter on` | Mute / unmute the bot |

  You can limit off/on to admins.
- **Safety.** Dry run is on by default: it renders clips on your PC and sends nothing. There's also a global cooldown, a per-person cooldown, an hourly cap, quiet hours, and a toggle to ignore its own messages. It also skips old messages when it starts, so it won't spam replies to chat history.
- **Self-healing.** It reloads the chat after 5 failed reads in a row. Diagnose shows what the bot can see and takes a screenshot.

## Setup guide (no coding needed)

### 0. Install once
1. Install **Python 3.11 or newer** from python.org. On Windows, tick **"Add python.exe to PATH"** on the first screen.
2. Download this project as a ZIP from GitHub and unzip it.
3. Double-click **`start_windows.bat`**. On Mac or Linux, run `./start_mac_linux.sh`.
   The first run takes about 3 to 5 minutes. It installs Chromium and FFmpeg by itself. Then the control room opens in your browser.

### 1. Make the burner account
1. Sign up for a new Facebook account in a private window. Use a real-looking name like "Walter Heisenberg" and a separate email.
2. Use it like a person for a day or two: add a profile photo, add friends from your group.
3. From your main account, add it to the group chat.

### 2. Give Walter reactions
- **Fast way:** press **Pick Walter picture** in step 1 and choose a render of your Walter model. That makes 8 starter clips.
- **Proper way:** make 2 to 4 second MP4s of your model for each reaction (laugh, facepalm, dance, rage, stare, confusion, dead, saymyname). Drag them onto the matching card under **Reactions**.
  - *VRoid Studio* (free): build Walter (bald head, goatee, glasses, black pork-pie hat). Export a VRM, then animate it in Blender, VSeeFace or Warudo and screen-record 3 seconds.
  - *Blender*: import the VRM with the VRM add-on, apply Mixamo animations (Laughing, Angry, Dancing, Facepalm), and render at 720p, 30 fps, H.264.
  - 720x720 or smaller keeps uploads fast.

### 3. Log in
1. Press **Open browser**. A Chromium window opens.
2. Log in as the **burner**, not your main account. Do any security checks. If it asks for your Messenger PIN, enter it.
3. You only log in once. The login is saved in `data/browser_profile/`. Keep that folder private, because it holds the session.

### 4. Point at the group chat
In that Chromium window, open the group chat, then press **Save chat**. Or paste the link. It looks like `https://www.facebook.com/messages/t/1234567890`.

### 5. Test, then go live
1. Type a message under **Try a message** to see exactly what Walter would send.
2. Press **Go live** with **Dry run** still on. Chat normally from your phone. The live log shows each message Walter read and why he did or didn't react.
3. Press **Diagnose chat** if nothing shows up. "rows" and "composer" should both be more than 0.
4. When the log looks right, switch **Dry run** off. Walter now posts for real.
5. Keep the PC awake. You can tick **Hide browser when live** after the first successful run.

### Commands for power users
```bash
python -m heisenbot                         # control room
python -m heisenbot run                     # listen with saved settings, no dashboard
python -m heisenbot test "HAHAHA fail"      # render one reply
python -m heisenbot starter walter.png      # build starter clips
python -m pytest -q tests                   # 30 tests, including Chromium runs
```

## Troubleshooting

| Problem | Fix |
|---|---|
| Diagnose says "Chat pane not found" | Open the group chat so the message box is visible, then press Diagnose again. |
| "rows 0" in Diagnose | Facebook changed its page layout. Open `data/config.json` and adjust `selectors`. The defaults use stable ARIA roles: `[role=row]`, `[role=textbox]`. |
| It reacts to its own posts | Its own bubbles are recognised because they sit on the right side. Keep the window at least 1000 px wide, or leave it headless. |
| No voice | edge-tts needs internet. Clips still send without the voice. |
| Account got a checkpoint | Stop the bot, clear the check by hand in the browser window, then lower `max_per_hour`. |
| Sender shows as "Someone" | Facebook sometimes hides names on grouped bubbles. The bot reuses the last name it saw. |

## Project layout
```
heisenbot/
  config.py      settings, defaults, validation (data/config.json)
  engine.py      trigger matching, cooldowns, commands, clip bank
  media.py       captions (Pillow), edge-tts voice, FFmpeg render, starter clips
  messenger.py   Playwright driver for facebook.com/messages
  bot.py         listen, decide, render, send loop, event log (data/logs/*.jsonl)
  server.py      local control room API (aiohttp)
  web/           dashboard UI
tests/           unit tests, plus a mock chat page for the Playwright test
deploy/          server installer, systemd units and timers, walter helper
.github/workflows/  ci.yml tests, deploy.yml push-to-deploy, watchdog.yml
clips/<reaction>/*.mp4
```

## Status
- Done: everything listed above. Tested here: automated tests pass, including a real Chromium session against a mock chat page. Clip rendering was checked by eye.
- Cloud mode: the password gate, remote browser (showing the real Facebook login page), login import/export, auto resume, render cleanup and the Cloudflare tunnel were all tested in a Linux sandbox. 30 tests pass. The installer, systemd units, health timer, auto restart and tunnel were checked on a systemd host.
- Not tested against live Facebook: I can't log into your account from here, and Facebook changes its page layout without notice. Diagnose and the editable selectors are there for when that happens.
- Possible next step: a local model (Ollama) that picks the reaction from the whole message instead of keywords.
