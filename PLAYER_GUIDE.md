# CKMud Client — Player Guide

Welcome! This is the official desktop client for **Dragonball Z: Celestial
Knights** (ckmud.com). Everything you need to get playing is in this folder.

## Getting started

There's no installer — just extract this zip anywhere you like (Desktop,
Documents, a games folder) and double-click **CKMud.exe**. Your settings,
characters, and scripts all live next to the exe, so the whole folder is
portable: copy it to a USB stick or another PC and everything comes with it.

On first launch, create a profile with your character name and password —
the client connects to ckmud.com and logs you in. You can add more
characters later and even play several at once: each one gets its own tab,
with its own scripts, settings, and look.

## Finding your way around

The terminal in the middle is the game. Around it: the **input bar** at the
bottom (your commands; up/down arrows recall history), the **chat panel**
with channel tabs (OOC, Clan, Tell, and friends — so chatter doesn't drown
in combat spam), **TOOLS** for everything scripting, and **SETTINGS** for
the client itself.

Scroll up in the terminal and a **LIVE** bar appears — new text keeps
flowing below while you read history. Click it to snap back.

## Make it yours: themes

Settings → Theme. Nine looks, from the classic gold-on-black to Synthwave
'84, Matrix Rain, and Dragon Ball. Themes are saved **per character** —
your Saiyan can run Synthwave while your Android runs Matrix, and the
client switches automatically when you change tabs.

Or build your own: pick **✦ Create / edit custom theme** in the theme list.
Three color wheels — backgrounds, accent, and prompt — and the client
derives a full matching palette while you watch it change live. Hit
Preview to tuck the wheels away and judge the result, name it, save it,
and it joins your theme list. Custom themes export as small files you can
share — a theme made on desktop looks identical on the phone and vice
versa.

## Scripting: aliases, triggers, timers

Open **TOOLS** and you'll find the script tabs. No programming needed for
the basics:

An **alias** turns a short command into a long one — make `easy` do your
whole power-up-and-hunt routine. A **trigger** watches the game text and
reacts — when the game says `You are thirsty.`, send `drink tears`. A
**timer** repeats a command on a schedule. Each editor has simple fields
(name, pattern, commands), an Enabled toggle, and a **Record** button that
captures what you do in-game and turns it into an alias for you.

When you outgrow the forms, the **Advanced Editor** shows the raw TinTin
source with syntax highlighting, find & replace, and live error checking.

**IMPORTANT — back up your scripts with the Export button.** Aliases,
triggers, and timers you create in these editors are saved in the client's
settings, *not* as files in the plugins folder. They survive updates on the
same device, but they will **not** follow you to a new computer or from
desktop to Android on their own. Hit **Export** in the Tools tabs every so
often — it writes your scripts to a file you can keep safe and **Import**
on any other device. (Scripts that live in `plugins\` as `.tt`/`.lua` files
just copy across like any file.)

## The plugins folder

The `plugins\` folder next to CKMud.exe is for shareable script files.
Drop in `.tt` (TinTin) or `.lua` files, hit **Plugins → Reload all** in the
client, and they're live. This is how you share scripts with clanmates —
one file, drop it in, done. Your imported and downloaded scripts stay put
through client updates.

Three levels of on/off, all in the Plugins window (hover any button
for a description): the **group ⏻** disables a whole folder, the
**file ⏻** disables one file, and **≡⏻** opens a list of every single
trigger and alias inside a file with its own switch — click a name to
preview its script, ✎ to open the file in the editor. Per-trigger
choices apply instantly, stick per character, survive re-imports, and
follow you via Profile sync.

## Importing Mudlet packages

If you have a Mudlet `.mpackage` (like the CK package), don't unzip it —
import it: **TOOLS → Plugins → Import from Mudlet…** and pick the file.
The client converts the whole package into a single plugin automatically.

Imported packages load in **SAFE mode**: any bots inside can't send a
single command until you arm them. Type `cksafe` to see the toggles —
`cksafe on` arms the master switch, then enable the specific bot you want
(e.g. `cksafe zetabot on`). Anything *you* type is never affected. Three
helper commands once a package is loaded: `cksafe` (bot switches),
`ckaliases` (every alias it added), `cktriggers` (every trigger, with
status).

CMUD `.pkg` files import the same way. Each CMUD class folder becomes
its own file (with `zsafe` class toggles), and triggers that weren't in
any class are sorted into `_root_a` … `_root_z` files so even a huge
flat CMUD tree stays browsable.

## Dock mode — your own capture windows (desktop)

If you came from CMUD, you probably had `#capture` windows — named
panes that collect specific lines (chat, quest spam, score output).
Dock mode brings them back:

1. **Settings → Dock mode** (per character). Off by default; with it
   off the client behaves exactly as before.
2. Play. The first time a script writes to a window — `#capture solar`,
   `#win ckstat ...` from an imported CMUD package, or
   `cecho("mywin", ...)` from a Mudlet one — that window's pane
   **creates itself** in place of the chat panel. No configuration.
3. Click **⚙** on any pane to tick what it shows: any capture window
   plus every chat channel (`OOC`, `Clan`, `Tell`, …). Want all chats
   in one window? Make a pane and tick them all. In dock mode chats
   are plain text lines — no bubbles, no tell dropdown.
4. Arrange with the mouse: drag a pane's **header onto another pane**
   to merge them; drag it to a pane's **edge** to split a new
   row/column; drag the **dividers** to resize; **×** closes a pane
   (its text returns to the terminal with the `[window]` prefix).
   Hover any pane button for a reminder of what it does.
5. **✎** renames a pane. **⚙ → Buttons…** adds your own buttons to it:
   one-click commands/aliases, or two-state toggles (label `Task`,
   ON-command `zsafe Tasks on`, OFF-command `zsafe Tasks off`) that
   show `[ON]`/`[OFF]` and light up — CMUD-style control buttons,
   with an adjustable size slider.
6. Your layout is remembered per character on this PC, and your
   routing (which windows show what) follows you to other PCs via
   Profile sync.

Anything you never route to a pane keeps printing in the terminal as
`[window] text`, so a half-set-up dock never loses anything. `#clr
<window>` clears that pane.

## Profile sync — take everything with you

Settings → **Profile sync**. Sign in once with your ckmud.com
website account and you can **Push** this device's scripts, channels,
themes and settings to your profile, then **Pull** them onto any other
device — phone to desktop and back. You pick exactly what to transfer
with checkboxes (device-specific stuff like font and joystick sizes
stays off unless you want it), and plugin files like CK.lua travel too.
Nothing syncs automatically — only when you press Push or Pull — and
your character passwords are never uploaded. Manage connected devices
on the website under Profile → Connected devices.

## Staying updated

The client checks for updates and the game announces new versions. When you
update, your `prefs.json`, `plugins\` folder, and characters are preserved —
only the program files are replaced. If anything ever looks wrong after an
update, your scripts are still in `plugins\` and your settings in
`prefs.json`; nothing is deleted.

Updates also back up the version they replace into a **"BU dont
delete"** folder. If a new build misbehaves, open **⬆ Update** and
click **⏪ Revert** — the app restarts on the previous version. The
updater then flags that exact build with a warning so you don't
re-install it by accident; updating becomes one click again as soon as
a newer build ships. Updates are always manual — nothing installs by
itself.

## Playing on your phone

There's an Android client too, with the same engine — scripts, triggers,
Mudlet/CMUD imports, themes, and multi-character play, plus touch extras
like the dual joystick and a hotbar. The Plugins window has the same
folder tree and per-trigger ≡⏻ switches as desktop, and your switch
choices sync between phone and PC. Ask in OOC for the current download
link.

## Stuck?

Type `help` in-game for game commands, or ask in **OOC** — someone's
always around. Welcome to Celestial Knights — now go channel your inner
Goku.
