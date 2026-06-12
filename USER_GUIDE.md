# CKMud — User Guide

A guided tour of the CKMud client for players. Covers both the **Android app** and the **Compose Desktop** app. Features that only exist on one platform are tagged **(Android only)** or **(Desktop only)**.

For scripting (aliases, triggers, timers, Lua, plugins), see [SCRIPTING_REFERENCE.md](./SCRIPTING_REFERENCE.md).

---

## What is CKMud?

CKMud is a native multi-session MUD client. It was built for CKMud (the Dragon Ball MUD at `ckmud.com:8500`) but works against any standard telnet MUD — generic-server users just toggle off the CKMud-specific features.

Both platforms share the same Kotlin engine (`:shared` module). What you learn about scripting, channels, and sessions is identical across Android and Desktop.

---

## Installing

### Android

Download the latest APK from [https://ckmud.com/files/ckmud.apk](https://ckmud.com/files/ckmud.apk) and install. The app will prompt you to allow installation from unknown sources the first time. After install, it auto-checks for updates on every launch — no Play Store required.

When a new version exists, you'll see a prompt on launch. Tap **Yes** to download, then **Install** when it completes. The app restarts itself cleanly when the update finishes.

### Desktop

Unzip the CKMud Portable folder somewhere. Run `CKMud.exe`. Settings persist next to the `.exe` in portable mode (look for `prefs.json`), or in `%APPDATA%\CKMud\` for a standard install.

---

## First Connection

You land on the **Character Select** screen. From here you can:

- **Add a character** — Stores name, password, and (optionally) host/port/MUD-name. Fill in everything you want auto-typed at the login prompt. The password is stored locally; on Android it's encrypted at rest via Android Keystore.
- **Quick Connect** — One-off connection without saving anything. Useful for poking at new MUDs.
- **Tap a saved character** — Opens the game screen and auto-connects.

Each character row is keyed by `(charName, host, port)`, so the same character on two different servers (e.g. CKMud main port `8500` vs builder port `8505`) gets two separate entries with separate scripts and settings.

---

## The Main Game Screen

### Top bar
- **Status text** — `Connected to host:port as YourName` / `Disconnected` / `Reconnecting in Ns...`
- **A− / A+** buttons — shrink/grow the terminal font (per-character).
- **TOOLS** button — opens the Tools drawer (aliases, triggers, timers, options, plugins, etc.).
- **≡ Drawer** button — opens the Chat / Stats / Affects / Hotbar tab drawer.

### Vitals bar (CKMud only)
Three progress bars: **PL** (power level), **Ki**, **Fatigue**. Driven by MSDP. Hidden on non-CKMud servers by default; toggle in Tools → Features → Vitals Bars.

### Combo + Opponent bar (CKMud only)
Above the terminal: current combo counter (e.g. `[Combo: 5]`) and the opponent's health percentage. Both are MSDP-driven and only appear when there's actually a combo or opponent. When a combo breaks, the bar briefly turns red showing **COMBO BREAK**, then fades out (~1.4 seconds total). The opponent bar disappears the moment the MUD stops reporting an opponent (e.g. enemy dies or you flee) — no animation, just hides.

### Terminal
The main MUD output area. ANSI colours are rendered. Scroll up to view history; the terminal automatically switches into **split-pane mode** with a divider, keeping the live tail visible at the bottom while you read the scrollback above. Tap the divider or scroll to the bottom to exit split mode.

**Selecting text** — long-press (Android) or click-drag (Desktop). A small action bar appears with Copy, Make Trigger, Make Alias, etc.

### Input bar
Type your command and hit Enter. Up/down arrows walk command history (filtered by prefix). The `↻` button toggles **Repeat Mode**: when on, pressing Enter on an empty line re-sends your last command.

### Joystick (Android)
Bottom-right corner. Two thumb wheels — left for movement (N/E/S/W/NE/NW/SE/SW), right for combat. Size and position are configurable per character (long-press the joystick).

### Hotbar
6 customisable slots above the input bar.

- **Desktop** — tap a filled slot to fire its command. Tap an empty slot to pick an existing alias to bind. Right-click any slot to edit label + command.
- **Android** — pin existing aliases into the slots (Tools → Hotbar tab → pick alias, or use the **Record New Alias** button to capture a sequence of commands live).

---

## Multi-Session

Up to **16 simultaneous connections**. Switch between them via the session tabs at the top (Desktop) or the session drawer (Android — tap **≡**).

Each session is fully isolated:
- Its own connection, terminal buffer (5000 lines), and prompt state.
- Its own scripts, channels, feature flags, hotbar, font size, etc. (keyed on `charName + host + port`).
- Background sessions still receive text and fire triggers — they just don't render the terminal until you switch to them.

### Persistent connection notification
While any session is connected, you'll see:
- A **summary notification** "CKMud Client — N sessions connected"
- One **child notification per session**: "[CharName] connected to [MudName]"

Tapping a child notification jumps you straight to that session.

### Keepalive
CKMud sends a harmless telnet NOP every 15 seconds to keep idle connections alive — your sessions won't time out from inactivity even if you're idle for hours.

---

## Channels

CKMud auto-detects standard MUD chat channels and tracks them in the **Chat drawer**.

### Default channels (14)
OOC, Chat, Quest, Event, Raid, Clan, Auction, Group, Tell, Say, Imm, Info, Gauntlet, Newbie.

Only a few are **visible** by default (OOC, Group, Say, Tell, Auction). Open Tools → Options → Channels to toggle visibility, change tab order, and pick which channels contribute to the chat-tab unread badge.

### Custom channels
You can add up to 11 of your own per session. Tools → Options → Custom Channels → Add. Provide a regex pattern that matches the channel format (case-insensitive). Example: `^\[Crew:` to capture a channel like `[Crew: Bob shouts hello]`.

### Per-channel pattern overrides (Mudlet-style)
If your MUD uses non-standard channel formats, override any channel's detection regex in Tools → Options → Channels → tap a channel → **Override pattern**. You can use named captures `(?<speaker>...)`, `(?<message>...)`, `(?<timestamp>...)` to teach the parser how to break apart speaker and text.

### Tell partner filtering
Open the Chat drawer, pick **Tell**. A second dropdown appears listing your recent Tell partners (up to 8 LRU), each with an unread count `(N)`. Pick a name to filter the message list to just that conversation; pick **(all conversations)** to see the mixed Tell timeline. Selecting a partner clears their unread badge.

The Tell channel itself still shows a total unread count across all partners.

### Channel command prefixes
Tools → Options → Channels lets you set a **send prefix** per channel:
- Empty (default) — chat input sends verbatim text.
- A command word like `ooc ` — input is prefixed before sending (typing `hello` in the OOC tab sends `ooc hello`).
- `-` (single dash) — read-only channel, no input bar.

For Tell, with a partner selected, input auto-prepends `tell <partner> ` so you can hold a conversation without retyping the name.

---

## Stats / Affects / Hotbar tabs (Drawer)

Open the bottom drawer (Android: **≡**; Desktop: bottom Hotbar button or via the right-side panel).

- **Chat** — channel tabs + message list (see above).
- **Stats** — character stats from MSDP (HP, Mana, level, etc.). Auto-refreshes.
- **Affects** — spells/effects currently on you, with remaining duration where the MUD reports it.
- **Hotbar** — your 6 quick-action slots.
- **Room** — current room info from MSDP.

Each tab can be toggled off individually in Tools → Features.

### Stats / Affects / Tracker popups (Desktop)
Desktop has dedicated popout windows for Stats, Affects, and the **Tracker** (XP/zenni/PL per hour rates). The Tracker auto-refreshes and shows current + peak rates so you can gauge grinding efficiency.

---

## Scrollback & Split-Pane

The terminal keeps the last **5000 lines** per session. Scroll up to read history.

When you scroll up, CKMud automatically splits the view:
- **Top pane** — frozen history snapshot, doesn't shift as new lines arrive.
- **Live tail** — bottom pane, keeps scrolling with incoming text.
- A divider strip labelled "LIVE VIEW" separates them.

This means you can read a long combat log without losing context to new prompts. Tap the divider or scroll the top pane back to the bottom to exit split mode.

Selecting text in the top pane copies it cleanly (no live-data interleaving).

---

## Recording Wizard — Capturing Aliases

Open the Tools drawer → **Aliases** tab → tap the **Record** button. A floating red **STOP** button appears over the terminal.

Now play normally — every command you type gets captured. When you have the sequence you want, tap STOP. A dialog asks for an alias name and short trigger word; the captured commands become the alias body, separated by `;`.

Example: To make an alias `go-arena` that does `west; west; north; enter arena`:
1. Start at your home location, hit Record.
2. Type `west`, `west`, `north`, `enter arena` one at a time.
3. Hit STOP. Name it `go-arena`, set trigger word to `goarena`.
4. From then on, typing `goarena` will execute all four commands.

(You can also use the **Make Alias** action on selected terminal text to seed an alias from one command directly.)

---

## Wait Widget

When a script issues a `#wait` directive (e.g. a buff rotation), a small floating pill appears in the corner showing the countdown for each pending wait. Tap to expand into a list of every active wait. Drag the pill to move it; its position is remembered per character.

You can hide brief waits in Settings → **Wait widget min seconds** (slider 0–30). E.g. set to 5 to only show waits longer than 5 seconds, so quick 1-second pauses don't clutter the screen.

---

## Notifications

When the app is backgrounded, notifications keep you informed:

- **Connection** — sticky summary plus per-session child notifications (tap to jump to that session).
- **Channel messages** — coalesced by channel when many arrive together; suppressed if the screen is off (no spam at 3am).
- **Own messages excluded** — your own OOC/Tell/Group messages don't notify you.
- **Update available** — when a new build is ready.

Configure which channels notify and which pop up over the game in Tools → Options → Channels.

---

## Auto-Buff (Desktop & Android)

If your MUD has buff/protection spells with predictable cast costs, CKMud can re-cast them automatically as they expire.

1. Enable **CK-Auto → Auto-Buff** in Settings.
2. Open the Auto-Buff skill picker.
3. Tick the skills you want maintained.
4. CKMud watches MSDP affects + your skill list. When a tracked skill drops, it queues a re-cast (with per-skill cooldown so you don't spam).

The implementation uses your character's known skill list (sent via MSDP) so it knows what's actually available.

---

## Dragon Ball Auto-Wish (CKMud only)

If the **Dragon Ball Auto-Wish** feature is enabled (Tools → Features), and you've picked a wish in Settings → CK-Auto → Dragonball Auto-Wish, CKMud will automatically make that wish the moment all seven dragonballs are in your possession (detected via MSDP).

---

## DPS Tracker (CKMud only)

Toggle on via Tools → Features → DPS Tracker. A floating panel shows your current and peak damage-per-second during combat. Resets between fights.

---

## Import / Export

Settings → **Export scripts** generates a JSON file containing all your aliases, triggers, timers, variables, and custom channels for the current character. Save it anywhere.

Settings → **Import scripts**:
- **Full import** — Pick an exported JSON. Replaces ALL current scripts and channels for that character. Use to migrate a character to a new device.
- **Partial import** (Android) — Pick an exported JSON, then a category (alias / trigger / timer). A multi-select dialog lets you choose which entries to import — handy for cherry-picking aliases from someone else's export.

Files use the format:
```
ckmud_<charName>_<host>_<port>_<YYYY-MM-DD>.json
```

See [SCRIPTING_REFERENCE.md](./SCRIPTING_REFERENCE.md#import--export-format) for the full JSON schema.

---

## Plugins

CKMud supports two plugin formats:

- **`.tt` files** — TinTin-style scripts. One file = many aliases/triggers/timers/variables in one bundle.
- **`.lua` files** — Lua scripts using the `mud.*` API.

### Default plugin folder
- **Android** — `Android/data/com.ckmud.client/files/plugins/`
- **Desktop** — `%APPDATA%\CKMud\plugins\` (or next to the `.exe` in portable mode)

Drop `.tt` or `.lua` files into the folder. Open Tools → **Plugins** to see what loaded; errors are shown per file.

### Custom plugin folder (Android: SAF)
Tools → Options → **Plugin folder** → pick any directory on your device (Downloads, Documents, Google Drive). The app uses Android's Storage Access Framework — your folder choice persists across reboots. The app copies the `.tt` and `.lua` files into its cache for reading and refreshes on every Reload.

### Custom plugin folder (Desktop)
Settings → **Plugin folder** → "Choose folder…" opens a directory picker. Choose any folder; settings persist via `PREF_PLUGIN_DIR_OVERRIDE`. Use **Reset to default** to revert.

For authoring plugins, see [SCRIPTING_REFERENCE.md](./SCRIPTING_REFERENCE.md#plugin-authoring).

### Updating scripts from a URL — `ckupdate`

If a script author (or you) hosts CKMud `.lua` files online — for example in a GitHub repo — you can pull updates without hunting for files. Type these in the input bar:

- `ckupdate set <url>` — remember where your scripts live, **once**. Use the repo's "raw" base URL, e.g. `https://raw.githubusercontent.com/you/repo/main/`. Saved permanently (in `~/.ckmud`).
- `ckupdate` — fetch everything listed in that source's `manifest.txt` (a plain list of filenames), drop it into your plugins folder, and reload live. No restart.
- `ckupdate where` — show the source you configured.
- `ckupdate <name> <url>` — one-off: grab a single file.

It only fetches already-CKMud-ready `.lua` files (it does not convert Mudlet packages), over http/https, capped at 5 MB. (Native/Desktop engine.)

---

## Feature Gates (Tools → Features)

11 toggleable features per character. Defaults: all ON for `ckmud.com`, all OFF for any other server.

| Feature | What it does |
|---|---|
| **Vitals Bars** | Show PL/Ki/Fatigue progress bars |
| **Combo History** | Combo counter + break notifications |
| **Opponent Bar** | Opponent health bar |
| **Quick Actions** | Fly/land/recall/etc buttons |
| **Default Channels** | Detect and log OOC/Chat/Quest/etc. **Disabling this also hides the default channel tabs in the Chat drawer.** |
| **Stats Tab** | Stats tab in drawer |
| **Affects Tab** | Affects tab in drawer |
| **Hotbar Tab** | Hotbar tab in bottom bar |
| **Dragonball Auto-Wish** | CKMud-specific automation |
| **Only For The Chosen** | Flame overlay easter egg |
| **DPS Tracker** | Floating damage-per-second panel |

If you're on a non-CKMud server, most of these are off by default — the UI declutters automatically.

---

## Settings Overview

### Global (apply to all characters)
- **Echo input** — Show your typed commands in the terminal output (matches old-school MUD client behaviour).
- **Repeat mode** — Empty Enter re-sends last command.
- **Collapse blank lines** — Squash consecutive blank lines. Auto-disables on ASCII-art-heavy regions to preserve formatting.
- **Keep screen on** (Android) — Prevent screen timeout while connected.
- **Stay connected when backgrounded** (Android) — Hold the socket open for ~30s after you leave the app, so quick checks don't drop you.
- **Diagnostic logging** — In-memory debug log viewable via View Logs (Tools menu). Default OFF.
- **Wait widget min seconds** — Hide brief `#wait` countdowns shorter than this threshold.
- **Theme** (Desktop) — Cyberpunk / Synthwave / Matrix / VT100 skin selection.
- **Bubble background intensity** (Desktop) — Slider for chat bubble tint strength.
- **Plugin folder** — Override the default `.tt`/`.lua` location.

### Per-character (apply only to current session)
- **Terminal font size** — Slider 9–24 sp.
- **Gag prompt** — Hide MUD prompt lines from the terminal (takes effect on reconnect).
- **Joystick sizes** (Android) — Tune left/right joystick scale.
- **Auto-Buff enabled** + skill list.
- **Dragonball Auto-Wish** selection.

---

## Tips

- **Long-press** anything for a hidden option (joystick to resize, Hotbar slot to edit, terminal line for actions).
- **Selecting text** in the terminal pops a quick action bar with **Copy / Make Alias / Make Trigger / Create Channel** — fastest way to script from live data.
- **`#help`** typed in the input bar opens the in-app command reference.
- **`#msdp`** dumps current MSDP state for debugging.
- **Switch sessions fast** — keep multiple characters connected and flip between them; triggers still fire in the background, so a Tell to your offline character still gets logged + notified.
- **Tools → View Logs** — if scripts misbehave, the diagnostic log shows trigger fires, alias expansions, and engine errors with timestamps.
- **Tools → Backup** — Auto-backup of your scripts is on by default. Find the rolling JSON snapshots in your storage; export manually for off-device safety.

---

## Troubleshooting

**Terminal looks weird after switching themes or fonts.**
Resize the window slightly (Desktop) or rotate the device (Android). The columns recompute on layout change.

**A channel isn't being detected on a non-CKMud server.**
The default regex assumes the CKMud chat format. Open Tools → Options → Channels → that channel → Override pattern, and supply a regex matching your MUD's format. Named captures `(?<speaker>...)(?<message>...)` work.

**Triggers fire but aliases don't expand inside trigger bodies.**
They do — aliases are expanded inline when the trigger body runs. Make sure the alias word is the FIRST token of the command. (Example: `flee; recall; easy` will expand `easy` as an alias.)

**Scripts on a new character look empty.**
Scripts are per-character per-server. Use Import → Full to copy them from another character.

**Custom channels don't capture until I reconnect.**
Should not happen on current versions. If you see it, file a bug — channel logs are supposed to initialise immediately on channel creation.

**The Wait widget shows brief 1-second waits I don't care about.**
Settings → Wait widget min seconds → drag the slider higher.

**I see `[guard] runaway send loop ... Lua sending PAUSED`.**
A Lua script got stuck in a loop flooding commands, so CKMud paused script-sending to stop the MUD from throttling or kicking you. Your own typed commands still work. Fix or remove the offending script (or just disconnect that plugin), then type `ckresume` to resume immediately — it also auto-resumes after 15 seconds. If it keeps happening, the culprit is usually a full Mudlet-package import; prefer a curated set of scripts via `ckupdate` instead.

---

## Where to go next

- **Power user?** [SCRIPTING_REFERENCE.md](./SCRIPTING_REFERENCE.md) — every TinTin directive, the Lua API, plugin authoring, regex tips.
- **Stuck on something specific?** The in-app **Tools → View Logs** is your friend for runtime diagnostics.
