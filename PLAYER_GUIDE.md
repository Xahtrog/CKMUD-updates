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

**One trigger, many lines.** A trigger can watch for several different
lines at once — click **➕ Add another pattern** in the trigger editor and
add as many as you want, then choose **fire when ANY of these match** (the
usual case) or **ALL of these** (only when every line is present). It runs
the same one set of commands no matter which line showed up — so a "combo
reset" that should fire on a combo break, a dodge, a daze ending, *or* a
new enemy is a single trigger instead of eight.

**Folders.** Sort triggers, aliases and timers into folders to keep big lists
tidy — set the **Folder** field in any editor, or pick several at once
(**☑ Select** on desktop, **long-press** on Android), tap **Move to…** and
choose a folder (or make a new one). Folders sync to your other devices with
everything else.

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

**Per-character vs. shared.** Each character gets its own sub-folder named
`<Character>@<server>` (for example `Gorthax@ckmud.com`). Files inside it
load **only for that character**. Anything left loose in the main
`plugins\` folder — or in any folder *without* an `@` in its name — is
**universal** and loads for every character. So a character's personal bot
goes in their folder, and shared clan scripts go in the root. New scripts
you make with **Plugins → New script** drop straight into the character
you're on, and you only ever see your own character's folder in the
Plugins window, never anyone else's.

Three levels of on/off, all in the Plugins window (hover any button
for a description): the **group ⏻** disables a whole folder, the
**file ⏻** disables one file, and **≡⏻** opens a list of every single
trigger and alias inside a file with its own switch — click a name to
preview its script, ✎ to open the file in the editor. Per-trigger
choices apply instantly, stick per character, survive re-imports, and
follow you via Profile sync.

**Folders, rename & move.** The Plugins window shows your folders — including
folders nested inside a character's folder — with short file names instead of
long paths. **Rename** or **delete** your own folders, **rename** files, and
**select several plugins** (long-press on Android) to **move** them into a
folder at once. Your character's `<Character>@<server>` folder is protected
from rename/delete, since that name is how the client tracks who owns the
plugins.

## Importing Mudlet packages

If you have a Mudlet `.mpackage` / `.xml` (like the CK package), don't unzip
it — import it: **TOOLS → Plugins → Import** and pick the file. An **import
wizard** then asks where it should go (**this character** or **all
characters**) and how to handle triggers: **Auto** turns simple "send"
triggers into native, **editable** CKMud triggers you'll find under
Tools → Triggers, while anything with real logic stays a plugin; or
**Keep as Lua** imports the whole thing as-is. Not happy with the result?
Delete it and re-import the other way.

Anything that stays a Lua plugin loads in **SAFE mode**: bots inside can't
send a command until you arm them. Type `cksafe` to see the toggles —
`cksafe on` arms the master switch, then enable the specific bot you want
(e.g. `cksafe zetabot on`). Anything *you* type is never affected. Helper
commands once a package is loaded: `cksafe` (bot switches), `ckaliases`
(aliases it added), `cktriggers` (triggers, with status).

CMUD `.pkg` files import the same way. Each CMUD class folder becomes
its own file (with `zsafe` class toggles), and triggers that weren't in
any class are sorted into `_root_a` … `_root_z` files so even a huge
flat CMUD tree stays browsable.

## Celestial Compendium (ckmud only)

A built-in item & mob guide. Click **📖 Compendium** (desktop top bar, or
the Android Tools drawer) and search the whole game world without leaving
the client — filter by type, tier, wear slot, zone, bosses or legendaries,
and open any entry for its stats, bonuses, special effects, and what drops
it (or, for a mob, what it drops). Names show in their real in-game
colours and rare gear is badged. Only available while you're connected to
the official ckmud game.

## Dock mode — your own capture windows (desktop)

If you came from CMUD, you probably had `#capture` windows — named
panes that collect specific lines (chat, quest spam, score output).
Dock mode brings them back:

1. **Settings → Dock mode** (per character). Off by default; with it
   off the client behaves exactly as before.
2. **Make a window — two ways:**
   - **By hand (no script needed).** With dock mode on and no windows yet,
     the empty dock shows **[ ➕ New window ]** (a blank pane you fill via ⚙)
     and **[ ➕ New chat window — all channels combined ]** (one pane
     pre-loaded with every chat channel). From any pane's **⚙** you can also
     pick **➕ New empty window** to add more.
   - **Automatically.** The first time a script writes to a window —
     `#capture solar`, `#win ckstat ...` from an imported CMUD package, or
     `cecho("mywin", ...)` from a Mudlet one — that window's pane **creates
     itself** in place of the chat panel.
3. Click **⚙** on any pane to tick what it shows: any capture window plus
   every chat channel (`OOC`, `Clan`, `Tell`, …). Want all chats in one
   window? Tick them all (or use **+ New chat window**). Want a channel in
   its **own** window? Click the **↗** next to it in the ⚙ list to break it
   out into a fresh pane. Chat lines keep their colours in dock mode but show
   as plain lines — no bubbles, no tell dropdown.
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

**Prefer a free-floating layout?** Turn on **Settings → Floating windows
(MDI)** instead. Now the terminal, your capture/chat windows, and the stat
panels (Stats / Tracker / Affects) each become a floating window you can lay
out exactly like an old CMUD profile.

Each window's title bar has: its **⚙** settings (right next to the name),
**A− / A+** to set that window's text size, **✎** rename (any window —
terminal, chat, stats), **⤒** bring to front, **⤓** send behind, **—**
minimise, **▣** maximise, **×** close (clicking any window also raises it).
**Move** a window by dragging its title bar (they can overlap); **resize**
from **any edge or corner** — the cursor turns into the Windows-style resize
arrows, so there are no handles to hunt for. Drag a window toward a screen
**edge or corner** to snap-tile it (left/right half or a quarter; dragging no
longer jumps to full-screen — use **▣** for that). Add windows from the
**＋ window** menu in the top-right.

A few per-window extras:

- **Chat windows** — make a window, open its **⚙**, and tick the channels
  (OOC, Clan, Tell, …) you want it to collect; **↗** beside a channel breaks
  it into its own window.
- **Button panels** — a window's **⚙ → 🔘 Buttons…** lets you fill it with
  one-click command buttons and on/off toggles. Toggles glow **green when
  ON** and **red when OFF**, and the buttons wrap to fill the window.
- **Stats your way** — a Stats window's **⚙** lets you tick which sections
  show (Combat / Attributes / Wealth), and **↗** breaks a section out into
  its own window.

The whole layout is saved per character. (Floating mode and the docked-pane
mode are independent — pick whichever you prefer in Settings.)

## Keybindings (desktop)

**Settings → Keybindings** turns any key into a shortcut. Click **Add
keybinding**, press the key (or a combo like **Ctrl+1** or **Shift+F2**),
confirm it, then choose what it does:

- **Custom command** — type any command to send.
- **Run an alias** — pick one of your saved aliases.
- **Toggle a timer** — pick a timer; the key flips it on/off.

Your shortcuts are listed right there, each with a **✕** to remove it, plus a
**Reset all** if you want to start over.

One rule keeps it simple: a **bound key always fires its action and never
types** into the command line. So bind keys you don't normally type as text —
the **numpad** is perfect (you type with the top-row numbers, not the numpad),
along with **F-keys** and **Ctrl / Alt** combos. Any key you *haven't* bound
types as normal. Keybindings are saved per character and travel with
**Profile sync**.

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

On **Android**, a **⬆ Check for updates** button sits right on the main
menu (and in Settings) — tap it any time and it tells you whether you're
up to date or offers the new build to download and install. If a newer
build is already waiting, the menu button says so.

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
