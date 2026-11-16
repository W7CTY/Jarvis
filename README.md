# Jarvis

A voice assistant for your Pixel. Say "Hey Jarvis" or long-press the power button, then talk. It answers out loud and acts for you: calls, texts, alarms, calendar, weather, live web answers, your phone's settings, and your AllStarLink node.

## Install

On your Pixel:

1. Open this link to download the latest version:
   **https://github.com/W7CTY/Jarvis/raw/main/apk/jarvis-1.2.0.apk**
2. Open the download. If Android blocks it, tap **Settings**, turn on **Allow from this source**, go back and tap **Install**. If Play Protect says it doesn't recognize the app, tap **Install anyway**. That's normal for apps from outside the Play Store.
3. Open Jarvis. It offers to delete the downloaded APK. Tap **CHOOSE APK** and pick the file.

## Updates

Jarvis checks for a new version twice a day. When one is out, a card appears on the main screen. Tap **INSTALL**, then confirm on Android's install screen. The first time, Android asks you to let Jarvis install updates. Your settings, keys and memories carry over.

To check right away, go to settings → UPDATES → **CHECK NOW**.

Jarvis installs an update only if it's a genuine Jarvis build signed with the same key as the copy you already have. A modified copy of the app won't run.

## First-time setup

Open Jarvis, tap the sliders icon (top right), and work down the list:

1. **Anthropic API key**: paste it and tap SAVE. Get one at console.anthropic.com. Jarvis can't think without it.
2. **Phone access**: tap each GRANT:
   - *Mic, contacts, calls, texts, calendar, location*
   - *Default assistant*: pick **Jarvis** under "Default digital assistant app". Long-press power then opens Jarvis.
   - *Display over other apps*: lets Jarvis open apps, set alarms and navigate when you start it hands-free
   - *Notification access*: lets Jarvis read your notifications to you
   - *Do Not Disturb access*
3. **Wake phrase**: turn on **Wake phrase "Hey Jarvis"**. No account or key is needed. Listening happens entirely on the phone, and a small notification shows while it's on. If it misses you or triggers by itself, adjust **Wake sensitivity**.
4. **About you**: your name and home city.

Keys are encrypted with the phone's hardware keystore. Once saved, they can only be replaced or cleared, never displayed again.

## Talking to Jarvis

Tap the glowing core, tap the mic, say "Hey Jarvis", or long-press power. You can also type.

- "What's the weather this weekend?"
- "Text Sarah I'm running 10 minutes late"
- "Call Mom"
- "Wake me up at 6:15"
- "Set a 12 minute timer for the pizza"
- "What's on my calendar tomorrow?"
- "Add dentist Thursday at 2pm"
- "Read my last three texts"
- "Any important notifications?"
- "Turn on the flashlight" / "Turn on Do Not Disturb" / "Media volume to 40 percent"
- "Navigate to Lucas Oil Stadium"
- "Who won the Colts game?"
- "Remember that my truck takes 5W-30"

Before any call, text or station action, Jarvis reads it back and waits for "yes" or a tap on CONFIRM. You can turn that off in settings, but it's safer on.

If Jarvis ends a reply with a question, the mic reopens so you can just answer. Tap the core or the stop button at any time to interrupt.

**On the lock screen:** by default Jarvis won't respond while the phone is locked, so nobody nearby can text or call as you. You can change this under *Respond while locked*.

## Your AllStarLink node

Jarvis talks to the node over the Asterisk Manager Interface (AMI) on your home network.

On the node, add a dedicated AMI user to `/etc/asterisk/manager.conf`. Use a long random secret, and change the `permit` line to match your LAN:

```ini
[jarvis]
secret = PUT-A-LONG-RANDOM-SECRET-HERE
deny = 0.0.0.0/0.0.0.0
permit = 192.168.1.0/255.255.255.0
read = system,call,command
write = system,call,command
```

Make sure `enabled = yes` in `[general]`, and that `bindaddr` isn't `127.0.0.1` (use the node's LAN IP or `0.0.0.0`). Then:

```bash
sudo asterisk -rx "manager reload"
# open AMI to your home LAN zone only, never the 44net zone
sudo firewall-cmd --zone=home --add-port=5038/tcp --permanent
sudo firewall-cmd --reload
```

In Jarvis settings → STATION, enter the node number, the node's LAN IP, port 5038, the user `jarvis`, and the secret.

Then try:

- "How's the node?"
- "Connect to 2000"
- "Monitor node 41223"
- "Disconnect from 2000" / "Disconnect all"

AMI sends its login in plain text, so this only works, and should only be used, on your home network.

## Your dashboards (WPSD, Control Center, anything with an HTTP API)

Under STATION → *Station endpoints*, add one line per thing you want Jarvis to be able to ask about:

```
wpsd_status | GET | http://192.168.1.60:8000/api/status | Hotspot mode, last heard and Pi health
node_dash   | GET | http://192.168.1.50:5000/api/status | Control Center status
```

Format: `name | GET or POST | URL | what it does`. Jarvis reads the description to know when to use each one. POST endpoints ask for confirmation first. Jarvis can reach only the URLs you list here.

## Privacy

- What you say is transcribed by your phone's Google speech service, then sent to Anthropic's Claude API along with recent conversation, your saved memories, and any tool results (for example, texts or notifications you asked it to read).
- Weather comes from Open-Meteo using your approximate location.
- Update checks contact GitHub. Nothing about you is sent.
- The wake word, keys, memories and conversation history stay on the phone and are excluded from Google backups.

## Changelog

**1.2.0**: New wake phrase "Hey Jarvis", built in and free. No Picovoice account needed.

**1.1.0**: Jarvis now updates itself from GitHub. Smaller, hardened app that refuses to run if it's been tampered with.

**1.0.0**: First release: voice and wake word, default-assistant support, phone control, calendar, texts, calls, weather, web answers, memory, AllStarLink node control, and custom station endpoints.
