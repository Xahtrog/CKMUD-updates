# CKMud — Scripting Reference

Full reference for the CKMud scripting engine. Covers TinTin-style directives, the Lua API, plugin authoring, and the import/export JSON schema.

For the player-facing walkthrough (channels, hotbar, settings, etc.), see [USER_GUIDE.md](./USER_GUIDE.md).

---

## Table of Contents

1. [Two scripting layers](#two-scripting-layers)
2. [Aliases](#aliases)
3. [Triggers](#triggers)
4. [Timers](#timers)
5. [Variables and substitution](#variables-and-substitution)
6. [TinTin directives — full reference](#tintin-directives--full-reference)
7. [Math (#math)](#math-math)
8. [Control flow](#control-flow)
9. [`#wait` — delays](#wait--delays)
10. [Repeat (`#N`) prefix](#repeat-n-prefix)
11. [Lua scripting](#lua-scripting)
12. [Plugin authoring](#plugin-authoring)
13. [Import / Export format](#import--export-format)
14. [Limitations & gotchas](#limitations--gotchas)

---

## Two scripting layers

CKMud has two coexisting script systems:

| Layer | Authoring | Best for |
|---|---|---|
| **TinTin** | Tools → Aliases/Triggers/Timers UI, or `.tt` plugin files | Quick combat aliases, simple triggers, MUD-side state (variables, math) |
| **Lua** | `.lua` plugin files via the `mud.*` API | Complex logic, conditional flows, anything where TinTin's directive syntax gets awkward |

Both run on the **same per-session script executor thread**.

> **Note:** the TinTin↔Lua shared-variable bridge (`mud.vars`) **is wired on the native Lua 5.4 engine** (the default). `mud.vars.target = "rat"` in Lua is readable as `@target` in TinTin and vice-versa; `getVariable`/`setVariable` are the function-style equivalents. See [Lua scripting](#lua-scripting).

### Precedence

When you type a command:
1. **Lua aliases** match first (highest priority).
2. **TinTin aliases** match next.
3. If nothing matches, it's sent to the MUD verbatim (after any `#N` repeat / `#wait` parsing).

When a line arrives from the MUD:
1. **TinTin triggers** run first.
2. **Lua triggers** run after.

Both fire if both match — there's no winner.

Aliases inside trigger bodies are also expanded — `flee; recall; easy` will expand `easy` as an alias if you have one.

---

## Aliases

An **alias** maps a short keyword to a sequence of commands. Useful for combat rotations, travel macros, "go home" shortcuts, etc.

### Fields
- **Name** — Display label for the Tools UI.
- **Alias word** — The trigger keyword (case-sensitive, must be the FIRST token typed).
- **Commands** — Body to execute. Split on `;` for multi-command sequences. Variables (`@var`), positional args from the typed input (`$1..$9` and `$*`), MSDP (`@msdp.KEY`), and inline `#directives` all work here.
- **Enabled** — Toggle without deleting.

### Example
| Field | Value |
|---|---|
| Name | `Energy Shield Combo` |
| Alias word | `esc` |
| Commands | `focus invigorate;focus energy shield;say ready` |

Type `esc` → fires all three commands.

### Aliases with positional args
If your alias body needs the words the user typed after the alias keyword, use `$1`..`$9` for individual whitespace-separated tokens, or `$*` for the whole thing joined back together as one string:

| Alias word | Commands |
|---|---|
| `kpk` | `kick $1;punch $1;kick $1` |

Type `kpk rat` → sends `kick rat`, `punch rat`, `kick rat`.

Use `$2`, `$3`, etc. for additional tokens (`$2` = the second word after the alias keyword, and so on). Use `$*` when you want everything-after-the-keyword as a single string — useful for things like `tellall $*` typed as `tellall hi everyone` sending `tellall hi everyone`.

**Important:** alias positional args use **`$N`**, but **trigger** captures use **`%N`** (see [Triggers](#triggers) below). The two systems do not share syntax — if you put `%1` in an alias body, it will be sent to the MUD as the literal text `%1`. If you put `$1` in a trigger body, same thing in reverse.

### From the recording wizard
Tools → Aliases → **Record** captures live commands into a new alias. See [USER_GUIDE.md → Recording Wizard](./USER_GUIDE.md#recording-wizard--capturing-aliases).

---

## Triggers

A **trigger** runs commands when a matching line arrives from the MUD.

### Fields
- **Name** — UI label.
- **Pattern** — Match expression (see below).
- **Commands** — Body, split on `;`. Captures from the match are available as `%1..%9`.
- **Enabled** — Toggle.
- **Case sensitive** — When off, case-insensitive matching.
- **Is regex** — When off (default), pattern is TinTin-style with literal text + `%N` wildcards. When on, pattern is a full Java regex (with `%N` still rewritten to `(.+?)`).

### TinTin pattern (default, `isRegex = false`)
Non-placeholder characters are literal. `%1`..`%9` are wildcards matching one or more of any character (non-greedy).

Examples:
| Pattern | Matches | Captures |
|---|---|---|
| `You see %1 here.` | `You see a fierce dragon here.` | `%1 = "a fierce dragon"` |
| `%1 has arrived.` | `Bob has arrived.` | `%1 = "Bob"` |
| `%1 tells you, '%2'` | `Alice tells you, 'hi'` | `%1="Alice"`, `%2="hi"` |

If your pattern has **no** `%N` placeholders and `isRegex=false`, the engine uses fast `String.contains()` instead of compiling a regex.

### Regex pattern (`isRegex = true`)
Full Java regex syntax. `%N` is still rewritten to `(.+?)` for convenience, so you can mix them with regex features.

Example:
```
pattern (regex): ^You gain (\d+) experience.
commands: #var lastxp %1;#note Just got @lastxp XP
```

Use named groups, lookaheads, character classes — anything Java's `java.util.regex.Pattern` supports.

### Common trigger patterns
| Goal | Pattern | Commands |
|---|---|---|
| Welcome new player | `^%1 has just connected.` | `say Welcome, %1!` |
| Auto-flee at low HP | `You feel weak!` | `flee` |
| Loot every kill | `is DEAD!!` | `get all corpse;get all from corpse` |
| Quote on death | `You have been killed by %1` | `tell %1 well played` |

### Trigger from selection
Select text in the terminal → action bar → **Make Trigger**. Pre-fills the pattern field with the selected text.

---

## Timers

A **timer** runs commands on a schedule.

### Fields
- **Name** — UI label, used by `#timer start/stop/toggle <name>`.
- **Interval (seconds)** — Integer seconds.
- **Type** — `repeat` (fires every N seconds, default) or `once` (fires once and stops).
- **Commands** — Body, split on `;`.
- **Enabled** — Off-state timers don't fire.

### Examples
| Name | Interval | Type | Commands |
|---|---|---|---|
| `quest tick` | 60 | repeat | `quest time` |
| `bid reminder` | 10 | once | `bid 50000` |
| `rest cycle` | 30 | repeat | `rest;#wait 25;stand` |

### `#timer` directive
From a script (alias body, trigger body, etc.):
```
#timer start questtick
#timer stop questtick
#timer toggle questtick
```

`#timer start` will also **auto-enable** a disabled timer.

---

## Variables and substitution

Variables are simple `name → string` pairs scoped to the session. They survive disconnects (live in `sessionVars` map) but are NOT persisted across app restarts unless you put them in a plugin file or set them from an alias body.

### Setting
```
#var target rat
#var {target} {a fierce rat}     ;; braces handle whitespace
#var target                      ;; show value
#var                             ;; list all vars
```

### Referencing
- `@target` — variable value, substituted into command/argument text.
- `@msdp.PL` — MSDP value (e.g. power level). Bare `@msdp` doesn't substitute; you need a `.KEY` suffix.
- `%1` … `%9` — regex capture groups from the **trigger** pattern that just fired. Only valid in trigger bodies; in alias and timer bodies, the captures list is empty and `%N` stays as literal text.
- `$1` … `$9` — positional args (whitespace-split words after the keyword) in an **alias** body. `$*` is all args joined as one string. Only valid in alias bodies.

### Substitution rules
1. `%N` captures are substituted first.
2. Then iterative passes (up to 10 rounds) of `@msdp.KEY` and `@varname`.
3. Unknown vars stay literal (`@unknownVar` would appear as text — useful for debugging typos).

Example chain:
```
#var weapon sword
#var attack {kick;slash with @weapon;kick}
#alias atk {@attack}    ;; (illustrative — aliases authored in UI, not via #alias)
```

Typing `atk` → expands `@attack` → expands `@weapon` inside → sends `kick`, `slash with sword`, `kick`.

### Math-storing variables
```
#math {count} {@count + 1}
```

See [Math](#math-math) for the full operator set.

---

## TinTin directives — full reference

All directives are case-sensitive and must start with `#`. Body args can be braced `{...}` (to include whitespace, nested directives, or `;`) or unbraced (read up to whitespace, only for non-final args).

### Directive list

| Directive | Syntax | Description |
|---|---|---|
| `#var` | `#var` / `#var name` / `#var name value` / `#var {name} {value}` | List, show, or set variables |
| `#math` | `#math {dest} {expr}` | Evaluate integer expression, store result as string in `dest` |
| `#if` | `#if {cond} {then}` / `#if {cond} {then} {else}` | Inline conditional (single-line) |
| `#if` / `#elseif` / `#else` | Multiple lines, see [Control flow](#control-flow) | Chained if/else |
| `#while` | `#while {cond} {body}` | Loop while cond is truthy (max 10,000 iters) |
| `#switch` | `#switch {value} { #case {x} {body}; #default {body} }` | First-match switch on string equality |
| `#case` / `#default` | Only inside `#switch` | Case clauses |
| `#break` | `#break` | Exit nearest `#while` or `#switch` |
| `#continue` | `#continue` | Skip to next `#while` iteration |
| `#foreach` | `#foreach {list} {varname} {body}` | Iterate list (split on `;` or `,`), body runs per element |
| `#format` | `#format {dest} {template} {arg1} ... {arg9}` | Sprintf-style: `$1..$9` in template substituted, `$*` = all args joined |
| `#note` | `#note text` / `#note {text}` | Print to local terminal (no MUD send). Substitutes vars first. |
| `#timer` | `#timer start/stop/toggle <name>` | Control a named timer |
| `#wait` | `#wait N` | Pause N seconds (fractional OK, e.g. `#wait 0.5`) |
| `#N` | `#N command` | Repeat `command` N times (cap 1000). See [Repeat prefix](#repeat-n-prefix) |

### Truthiness (for `#if` and `#while`)
A condition is **true** if:
- Numeric and non-zero: `1`, `42`, `-1` are true; `0` is false.
- Non-numeric and non-empty: `"hello"` is true.
- Empty string is false.

After substitution, `#if {@hp}` is true if `@hp` is set to anything other than `"0"` or `""`.

For numeric comparisons, use `#math`:
```
#math {below50} {@hp < 50}
#if {@below50} {flee;#note Low HP — fleeing!}
```

### Comparison operators inside `#if` / `#while`

Supported operators in condition expressions: `==`, `!=`, `<`, `>`, `<=`, `>=`. Combined with `&&` (and) and `||` (or). Parentheses respected for grouping. Lowest precedence first: `||`, then `&&`, then comparisons.

| Operator | Behaviour |
|---|---|
| `==`, `!=` | **Tries numeric first; falls back to case-sensitive string equality** if either side isn't a valid integer expression. |
| `<`, `>`, `<=`, `>=` | **Integer-only.** Non-numeric operands silently make the condition false (no lexicographic string ordering). |

This means trigger captures can be compared directly to literal words:

```
Trigger pattern: You have combined components: %1 %2 %3 %4
Trigger body:
  #if {%1 == legendary && %2 == perfect} {put %3 awesome}
  #else {put %2 combined}
```

Captures are substituted before the condition is evaluated. After `%1` becomes `legendary` and `%2` becomes `perfect`, `evaluateLeaf` sees `legendary == legendary` — tries integer eval, fails (non-numeric), falls back to string equality, returns true. The other branch runs only when one of the words doesn't match.

**Gotchas:**
- String equality is **case-sensitive**. `Legendary != legendary`.
- Numeric semantics still take priority when BOTH sides are valid integers — `5 == 05` is true (both eval to 5).
- `<`, `>`, etc. on strings return false. Use `==` / `!=` for word matching.
- Variables: `#var x foo; #if {@x == foo} {...}` works (string fallback). `#var n 5; #if {@n == 5} {...}` also works (numeric path).

### Body splitting
Commands in a body separated by `;` are dispatched in order:
```
#if {@hp < 100} {sip health;rest;stand}
```

To include a literal `;`, brace the segment that contains it, or escape via inner braces:
```
#note {Use ; as a separator}
```

---

## Math (`#math`)

`#math {dest} {expr}` evaluates `expr` as a 64-bit integer and stores the result as a string in variable `dest`.

### Supported
- `+`, `-`, `*`, `/`, unary `-`.
- Parentheses for grouping.
- Variable references: `@varname` looks up the variable; if missing or non-numeric, treated as `0`.
- Capture refs: `%1`..`%9` are substituted if `#math` is invoked from inside a **trigger** body (where the captures are the regex matches). In alias and timer bodies the captures list is empty, so `%N` stays literal — use `$N` instead inside alias bodies (e.g. `#math {total} {@total + $1}`).
- Division by zero: returns `0` (no crash).

### Examples
```
#math {sum} {2 + 3 * 4}              ;; 14
#math {par} {(2 + 3) * 4}            ;; 20
#math {hp_pct} {@hp * 100 / @maxhp}  ;; uses @hp and @maxhp vars
#math {after} {@before + %1}         ;; %1 captures only resolve inside trigger bodies
#math {gained} {@before + $1}        ;; $1 is the first arg in an alias body
#math {next} {@count + 1}
```

### Integer-only
There is no float math. `3 / 2` returns `1`. If you need floats, do them in Lua (see [Lua scripting](#lua-scripting)).

---

## Control flow

### `#if` / `#elseif` / `#else` (multi-line)

```
#if {@hp < 30} {
    sip mega-heal
    quaff prayer
} #elseif {@hp < 60} {
    rest
} #else {
    #note HP fine
}
```

First true branch runs. Inline 2- and 3-arg forms work too:
```
#if {@target} {kill @target} {#note No target!}
```

### `#while`

```
#var i 0
#while {@i < 5} {
    #note tick @i
    #math {i} {@i + 1}
}
```

Hard cap: **10,000 iterations** per `#while` invocation (defends against runaway loops). `#break` exits the loop, `#continue` skips to the next iteration.

### `#switch` / `#case` / `#default`

```
#switch {@target_type} {
    #case {newbie} {tell @target_name go play with rats}
    #case {boss}   {focus invigorate;focus energy shield}
    #default       {#note Unknown target type: @target_type}
}
```

Match is **literal string equality** after substitution (no Lua-pattern or regex). First match wins; no fallthrough; `#default` runs if no `#case` matched. `#break` is honoured.

### `#foreach`

```
#var enemies {Goku;Vegeta;Piccolo}
#foreach {@enemies} {e} {
    consider @e
}
```

Splits the list on `;` **or** `,`. The body runs once per element with the temp variable (`@e` in the example) set to that element.

### `#break` / `#continue`

Throw control exceptions caught by the nearest enclosing `#while` (or `#switch` for `#break`). Outside any loop/switch they're silently no-ops.

### `#format`

Sprintf-like, but using `$1..$9` placeholders and `$*` for all args:
```
#format {greeting} {Hello $1, you have $2 zenni} Bob 5000
#note @greeting
;; → Hello Bob, you have 5000 zenni

#format {csv} {$*} alice bob carol
#note @csv
;; → alice bob carol
```

---

## `#wait` — delays

```
#wait 5      ;; wait 5 seconds
#wait 0.5    ;; wait 500ms (fractional)
```

**`#` prefix is required.** Bare `wait 5` will be sent to the MUD as a literal command — this is intentional (matches strict TinTin convention). Old scripts using bare `wait` were migrated by a one-time shim; new scripts must use `#wait`.

`#wait` works inside alias bodies, trigger bodies, and timer bodies. The remaining commands in the body are queued and dispatched after the wait elapses.

**Limitation:** `#wait` does **not** currently pause the body of a `#while` loop. The wait directive is consumed at the body-scheduler level. If you need delays inside a loop, restructure as a `#timer` or a recursive trigger pattern.

### Wait widget
Each active `#wait` registers in a per-session WaitRegistry. The floating wait widget (see [USER_GUIDE.md](./USER_GUIDE.md#wait-widget)) shows live countdowns for all pending waits.

---

## Repeat (`#N`) prefix

Prefix any command with `#N` where N is 1–1000 to repeat it that many times:
```
#10 north
#3 sip
#1000 east     ;; only valid if you've got somewhere to go
```

Per-session — repeats queued on session A don't fire on session B (the engine checks `isActive` before sending each iteration). Capped at 1000 per CLAUDE.md.

You can also enable **Repeat Mode** in settings (the `↻` button next to Enter) — when on, pressing Enter on an empty input field re-sends your last command. Useful for spam-killing or repeated `look`s.

---

## Lua scripting

CKMud embeds **native Lua 5.4.7** — the real reference interpreter from lua.org, compiled from source for Android (arm64/arm/x86_64) and Windows. Each session has its own sandboxed Lua state that runs on the same script-executor thread as TinTin scripts.

Because this is genuine Lua 5.4 (not a JVM re-implementation), the full modern language works: integer/float distinction, integer division `//`, bitwise operators (`&`, `|`, `~`, `<<`, `>>`), `goto`, `<const>` and `<close>` variables, and the 5.4 standard library.

> **Fallback:** if the native library fails to load on a device, the session falls back to the legacy LuaJ engine (Lua 5.2, JVM-based) and prints a `[Lua54]` note to the terminal. On the fallback engine, 5.4-only syntax will not parse. A healthy install always runs 5.4.

Drop `.lua` files into your plugin folder (see [Plugin authoring](#plugin-authoring)) and they're loaded on app start or via Tools → Plugins → Reload.

### `mud.*` API

| Call | Args | Returns | Description |
|---|---|---|---|
| `mud.send(cmd)` | string | nil | Send raw command to the MUD (same as typing into input bar but bypasses alias matching) |
| `mud.note(text)` | string | nil | Print local line to terminal (no MUD send) |
| `mud.msdp(key)` | string | string or nil | Read MSDP value (e.g. `mud.msdp("PL")`) |
| `mud.vars` | table | — | **Wired on native Lua 5.4.** Bridged TinTin variable namespace — `mud.vars.x = "y"` ↔ `@x`. Values coerced to strings; setting nil deletes. `getVariable(k)` / `setVariable(k, v)` are equivalents. |
| `mud.trigger(pattern, fn)` | Lua pattern (or PCRE) + function | id (number) | Register a trigger. `fn(line, cap1, cap2, ...)` per Mudlet convention. Global `matches[]` populated (`matches[1]` = whole line/match, `matches[2..]` = captures). Auto-uses PCRE if the pattern contains `(?` or `\`. |
| `mud.triggerRegex(pattern, fn)` | PCRE + function | id | Force a Java/PCRE-regex trigger. Supports named groups `(?<name>)` → `matches.NAME`, lookaround, backrefs, alternation. |
| `mud.alias(name, fn)` | string + function | id | Register an alias. `fn(rest_of_input)`. Matches on the literal keyword extracted from `name` (so a PCRE-shaped name like `strike(?: ...)?` still matches `strike rat`); populates `matches[2..]` from the typed words. |
| `mud.timer(seconds, fn)` | number + function | id | One-shot timer |
| `mud.tick(seconds, fn)` | number + function | id | Repeating timer (every N seconds) |
| `mud.cancel(id)` | number | nil | Cancel a timer or tick by id |

Debug toggles (native 5.4 engine): set `mud.debug_aliases = true`, `mud.debug_triggers = true`, or `mud.debug_timers = true` from any script to print registration/dispatch diagnostics to the terminal.

### Lua patterns ≠ regex

Lua uses its own pattern syntax (smaller and simpler than regex):
- `%d` digits, `%a` letters, `%w` alphanumerics, `%s` whitespace, `%p` punctuation
- `%D %A %W %S %P` are their inverses
- `+` one or more, `*` zero or more, `-` lazy match
- `(...)` captures
- `.` any character
- No alternation `|`, no lookaheads, no `\d{2,4}` quantifiers — use the simpler Lua equivalents

Example: matching `You gain 1234 experience`:
- **Lua pattern:** `^You gain (%d+) experience`
- Java regex would be: `^You gain (\d+) experience`

### PCRE / named-capture triggers

When a Lua pattern isn't enough, use full Java/PCRE regex via `mud.triggerRegex`, or just write PCRE in `mud.trigger` — it **auto-detects** PCRE when the pattern contains `(?` or a backslash and routes it through `java.util.regex`.

PCRE unlocks named groups, lookaround, backreferences, alternation, and `\d \w \s` classes. Captures arrive **both** positionally (`function(line, cap1, ...)` and `matches[2]`, `matches[3]`...) **and** by name in `matches`:

```lua
mud.triggerRegex([[^\[Kaioken: (?<CURR>\d+)/(?<MAX>\d+) ]]], function(line, curr, max)
    -- positional: curr, max  (also matches[2], matches[3])
    -- named:      matches.CURR, matches.MAX
    mud.note(("Kaioken %s / %s"):format(matches.CURR, matches.MAX))
end)
```

Double backslashes in a normal Lua string (`\\d`), or use a `[[long-bracket]]` literal to avoid escaping. `matches[1]` is the whole matched text for PCRE triggers.

### `multimatches[]` (Mudlet multi-line compat)

`multimatches[n]` returns the **current** `matches` table for any `n`, so a converted Mudlet handler that reads `multimatches[2][2]` (pattern-line 2, first capture) resolves to `matches[2]`. CKMud doesn't do true cross-line multi-line triggers; this shim covers the common anchor-line + capture-line idiom.

### `ckupdate` — pull script updates from a URL

Built-in updater for **already-converted CKMud `.lua`** files (it does *not* convert Mudlet packages):

```
ckupdate set <repo-raw-base-URL>   -- remember a source once (saved to ~/.ckmud)
ckupdate                            -- pull every file in <base>/manifest.txt, reload live
ckupdate where                      -- show the configured source
ckupdate <name> <url>               -- one-off single-file install
```

`manifest.txt` is a newline list of filenames. Filenames are sanitised (no path traversal, forced `.lua`); http/https only; 5 MB cap; native engine only.

### Runaway-send guard

If a Lua script floods the socket (>~30 sends/second) the engine **pauses Lua-initiated sending** and prints `[guard] runaway send loop ... PAUSED`, so the MUD doesn't throttle/boot you. Typed commands still work (different path). Fix or unload the script, then type **`ckresume`** (also auto-resumes after 15s). A timer firing every second or two, or a modest burst, never trips it.

### Example plugin

```lua
-- buff_rotator.lua

-- Re-cast Energy Shield when it expires
mud.trigger("^Your Energy Shield fades.", function()
    mud.send("focus invigorate")
    mud.send("focus energy shield")
    mud.note("Re-buffing Energy Shield")
end)

-- Remember the current target across aliases (plain Lua state)
local target = "rat"

mud.alias("settarget", function(rest)
    if rest and #rest > 0 then target = rest end
    mud.note("Target is now: " .. target)
end)

-- "kk [target]" alias = double kick
mud.alias("kk", function(rest)
    local t = (rest and #rest > 0) and rest or target
    mud.send("kick " .. t)
    mud.send("kick " .. t)
end)

-- Quest tick reminder every 5 minutes
mud.tick(300, function()
    mud.send("quest time")
end)

-- One-shot delayed action
mud.alias("wakeup", function()
    mud.timer(10, function() mud.send("stand") end)
    mud.send("rest")
    mud.note("Resting for 10 seconds, then standing.")
end)
```

### Sandbox

The native Lua 5.4 state opens the full standard library, then locks down anything that touches the host system before any user script runs:

- `io` — removed entirely (no file reads/writes).
- `os` — `execute`, `exit`, `remove`, `rename`, `tmpname`, `getenv` removed. `os.time`, `os.clock`, `os.date`, `os.difftime` are kept (read-only utilities, handy for scripts).
- `package` — `loadlib`, `cpath`, and the searchers are removed (no native code loading). `require(name)` exists but returns a benign auto-vivifying stub table (so Mudlet-style cross-module `require` calls don't crash); it does not load files.
- `debug` — write-side functions (`sethook`, `setlocal`, `setupvalue`, `debug.debug`, …) removed. Read-side (`traceback`, `getlocal`, `getinfo`) kept.
- `print` is rerouted to the CKMud terminal as a local note.

You cannot read/write arbitrary files, spawn processes, or load native code — Lua plugins are safe to share. Persistent storage is available through the sandboxed `db` store (see the Mudlet compatibility section), which saves to a per-character file managed by the client.

### Reset semantics

When the plugin folder is reloaded (Tools → Plugins → Reload), all registered triggers/aliases/timers are wiped before re-running the `.lua` files. This means you can iterate without restarting the app, and ensures stale registrations don't pile up. Built-in shim commands (`ckaliases`, `ckupdate`, `ckresume`) are **preserved** across reloads — only plugin-registered handlers are cleared. (Note: `ckupdate` itself triggers a reload after fetching a file.)

---

## Plugin authoring

### `.tt` files (TinTin)

A `.tt` file is a flat list of TinTin-style directives that get parsed once at load and converted into Aliases / Triggers / Timers / Variables in the user's script store. The runtime CommandProcessor is not involved at parse time — the parser builds data objects directly.

Supported directives (more lenient than the runtime — designed for portability with other clients' plugin formats):

| Directive | Aliases | Syntax | Maps to |
|---|---|---|---|
| `#alias` | `#al` | `{keyword} {commands}` | Alias |
| `#action` | `#act` | `{pattern} {commands}` | Trigger (regex, case-insensitive) |
| `#tick` | — | `{name} {commands} {interval-seconds}` | Timer (repeat) |
| `#variable` | `#var` | `{name} {value}` | Variable |
| `#nop` | — | `{anything}` | Comment |

**Recognised but silently ignored** (for paste-from-other-clients compatibility): `#substitute`/`#sub`, `#highlight`/`#high`, `#gag`, `#path`, `#config`, `#info`, `#help`, `#session`/`#ses`. They don't error but they don't do anything yet.

**Comment styles:**
- Lines starting with `/` (after trim)
- Lines starting with `;`
- `#nop {whatever}` lines

**Arg syntax:**
- `{braced args}` allow whitespace, nested braces, and `\\{` `\\}` escapes
- Unbraced args read up to whitespace (only for non-final positional args)
- Final arg can be unbraced and read to end-of-line

**Unknown directives** produce a per-file error (visible in Tools → Plugins). The file still loads with the recognized directives.

### Example `.tt`

```
;; combat.tt — basic combat helpers
/ Also can comment with leading slash

#nop {Aliases}
#alias kk {kick @target;kick @target}
#alias ff {flee;flee;flee}

#nop {Triggers}
#action {You are out of mana.} {sip mana;sip mana}
#action {%1 has DIED!} {get all corpse;loot corpse}

#nop {Timers}
#tick {quest_tick} {quest time} {60}

#nop {Variables}
#variable {target} {rat}
```

### `.lua` files

Plain Lua files. The whole file runs once at load (it's `dofile`-equivalent inside the sandbox). Calls to `mud.trigger`/`mud.alias`/`mud.timer`/`mud.tick` register handlers; calls to `mud.send`/`mud.note`/`mud.msdp` run at load too if you put them at top level.

A typical plugin file has registrations at the top and helper functions below. See [Lua example plugin above](#example-plugin).

### Reload

Tools → Plugins → **Reload** does:
1. Reset Lua VM (wipes all Lua handlers).
2. Re-scan plugin folder (flat — subdirectories NOT recursed).
3. Re-parse every `.tt` file, replacing the previous plugin scripts.
4. Re-run every `.lua` file in the sandboxed VM.

Errors are surfaced per-file in the Plugins dialog.

### Where to put plugin files

See [USER_GUIDE.md → Plugins](./USER_GUIDE.md#plugins) for default folders and how to override.

**Plugin scripts** live alongside your manually-authored aliases/triggers/timers — both contribute to matching. Plugin scripts are tagged internally so a Reload wipes them but leaves your hand-authored scripts intact.

---

## Import / Export format

CKMud exports a single JSON file containing scripts + custom channels for one character.

### Wire format

```json
{
  "format": "ckmud-export-v1",
  "exportedAt": "2026-05-29T16:38:00Z",
  "appBuildName": "1.617",
  "character": {
    "name": "Your_NAME_HERE",
    "host": "ckmud.com",
    "port": 8500
  },
  "scripts": {
    "aliases": [
      {
        "id": "uuid-here",
        "name": "Energy Shield Combo",
        "alias": "esc",
        "commands": "focus invigorate;focus energy shield;say ready",
        "enabled": true
      }
    ],
    "triggers": [
      {
        "id": "uuid-here",
        "name": "Auto-flee low HP",
        "pattern": "You feel weak!",
        "commands": "flee",
        "enabled": true,
        "caseSensitive": false,
        "isRegex": false
      }
    ],
    "timers": [
      {
        "id": "uuid-here",
        "name": "quest tick",
        "intervalSec": 60,
        "type": "repeat",
        "commands": "quest time"
      }
    ],
    "variables": [
      { "id": "uuid-here", "name": "target", "value": "rat" }
    ]
  },
  "customChannels": [
    {
      "id": "uuid-here",
      "name": "Crew",
      "patternString": "^\\[Crew:",
      "createdAt": 1717084680000,
      "enabled": true
    }
  ]
}
```

### Field notes

- `id` — UUIDs are regenerated on import (avoids collisions). You don't need to keep them unique across exports.
- `enabled` — defaults to `true` on import if omitted.
- `caseSensitive` — defaults `false`.
- `isRegex` — defaults `false`.
- `type` (timer) — `"repeat"` or `"once"`.
- `patternString` (custom channel) — raw regex source (case-insensitive flag is applied automatically at compile time).
- **Timer `enabled` quirk** — the exporter currently does not write the `enabled` field for timers, so all exported timers re-import as enabled. (Tracked separately; not a problem in practice since you can toggle in the UI.)

### Suggested filename

```
ckmud_<charName>_<host>_<port>_<YYYY-MM-DD>.json
```

Generated by the export dialog.

### Partial import (Android)

The Android **Import** flow lets you pick a category (alias/trigger/timer) and multi-select which entries to bring over. It looks at the same JSON's `scripts.aliases` / `scripts.triggers` / `scripts.timers` arrays. Useful for sharing single triggers without overwriting your whole script set.

### Full import (both platforms)

Replaces ALL scripts + custom channels for the target character. Confirm dialog appears first. Use this for character migration.

---

## Limitations & gotchas

This section is the "you'll thank yourself later" list. Read it before authoring big plugins.

### TinTin engine

1. **`#wait` doesn't pause `#while` loops.** Wait directives are consumed at the body scheduler, not in `process()`. Use `#timer` or recursive triggers instead.
2. **`#switch` cases are literal string match only** (no Lua-pattern / regex).
3. **Math (`#math`) is integer-only** (64-bit `Long`). `3 / 2 = 1`. Do float math in Lua. Note: `==` and `!=` inside `#if` conditions DO support string comparison as a fallback when either side is non-numeric — see [Comparison operators](#comparison-operators-inside-if--while).
4. **Math `/0 = 0`** — no exception, no NaN. You won't see crashes, but you might see surprising zeros.
5. **Substitution caps at 10 passes.** Pathologically nested `@vars` referencing each other will stop substituting and leave `@name` literal.
6. **`#var` is the only directive for variable assignment.** TinTin's `#variable` syntax is recognised in `.tt` plugin files but not at runtime — use `#var` in alias/trigger bodies.
7. **Pattern wildcards `%N` are non-greedy.** Mostly what you want. If you need greedy, use regex mode with `(.+)`.
8. **Trigger pattern `%N` and alias `%N` share the same substitution machinery.** Both are filled from the current execution context's captures.
9. **Bare keyword directives that TinTin++ has aren't implemented at runtime**: `#alias`, `#unalias`, `#trigger`, `#untrigger`, `#send`, `#echo`, `#read`, `#write`, `#list`, `#function`, `#replace`, `#substitute`, `#highlight`, `#gag`, `#path`, `#config`, `#info`, `#log`, `#zap`. Use the Tools UI or `.tt` plugin format for authoring instead.
10. **Helper commands intercepted by the app, not the engine**: `#help`, `#msdp`, `#msdp connect`, `#msdp disconnect`. They work as one-line typed commands but won't run from inside script bodies.

### Lua engine

1. **Use Lua patterns, not regex.** `%d+` is right, `\d+` is wrong.
2. **VM is reset on plugin reload.** Don't rely on Lua module-level state across reloads.
3. **No `io`, no `os.execute`, no native libs.** Sandbox is strict (see [Sandbox](#sandbox)).
4. **`mud.vars` works on the native Lua 5.4 engine** and shares the `@var` namespace with TinTin (`getVariable`/`setVariable` too). Plain Lua locals are still best for file-private state; use `db` for persistence across reloads.
5. **Trigger callback receives the whole line in arg 2 if there were no captures.** Most of the time you have captures; just be aware.
6. **Bad patterns don't kill the engine.** A pattern that errors at match time prints a `[TRIG-PATTERN-ERR]` note identifying the trigger id and pattern; other triggers keep running.
7. **MSDP values are strings.** `tonumber(mud.msdp("PL"))` before arithmetic.

### Triggers

1. **Patterns without `%N` use `String.contains()`** (fast path). This means partial matches succeed. Use regex mode with `^` anchors if you need start-of-line match.
2. **ANSI escapes are stripped before matching.** Your pattern should match the visible text, not the raw bytes.
3. **Trigger bodies expand aliases inline.** The first token of each `;`-split body is checked against aliases.
4. **Disabled triggers don't fire.** Toggle via the Enabled column.

### Plugins

1. **Subdirectories are NOT recursed.** Drop your `.tt` and `.lua` files flat in the plugin folder.
2. **`.tt` and `.lua` files are loaded in alphabetical order.** Use a numeric prefix (e.g. `00-shared.lua`, `10-combat.lua`) if loading order matters.
3. **Plugin scripts and user-authored scripts coexist.** Reload only wipes plugin entries.
4. **Unknown `.tt` directives error but don't block the file** — the rest of the file still loads.

### Multi-session

1. **All scripts are per-character per-server.** `(charName, host, port)` tuple. Two sessions for the same character on different ports → two script sets.
2. **Triggers fire in background sessions.** A trigger you wrote on character A still fires when you're focused on character B.
3. **Repeat (`#N`) commands are isolated** — they check the originating session's `isActive` flag and only send if still active.
4. **Cap of 16 sessions** enforced at the SessionManager.

### Platform differences

1. **Hotbar storage diverges between platforms** — Desktop stores raw (label, command) per slot; Android stores pinned alias IDs that resolve against ScriptStore. Exports/imports don't transfer hotbar slots.
2. **Plugin folder picker** uses Android's SAF (`content://` URIs) on Android, `JFileChooser` on Desktop.
3. **Stats / Affects / Tracker popups** are Desktop-only as separate windows; Android shows them in drawer tabs.
4. **Joystick** is Android-only.

---

## Mudlet compatibility

CKMud's native Lua 5.4 engine ships with a built-in Mudlet API shim so scripts written for Mudlet can run with few or no changes for the supported subset of functions.  Drop a Mudlet `.lua` script straight into your plugins folder and the engine recognises the Mudlet-style function calls and routes them through the equivalent CKMud APIs.

> **Scope honesty:** the shim covers a large, useful subset — it is not all of Mudlet. Functions sort into three tiers: **real** (full behaviour), **accept-and-ignore** (callable, returns a plausible value, does nothing — so scripts don't crash), and **absent** (calling them errors). The tables below say which is which. If you're writing *new* scripts rather than porting, prefer the `mud.*` API — it's the documented, fully-supported surface.

### Real implementations

These Mudlet globals do what Mudlet scripts expect:

| Function | Notes |
|---|---|
| `send(cmd)`, `sendAll(...)` | Identical to `mud.send(cmd)`. |
| `echo(text)`, `display(v)`, `print(...)` | Routed through `mud.note`. |
| `cecho(text)`, `decho(text)`, `hecho(text)` | Routed through `mud.note`.  Mudlet inline colour tags (`<red>`, `<255,128,0>`, `<#ff8800>`, `<red,blue>`, `<reset>`) are **stripped** — the text shows up uncoloured. |
| `tempTimer(seconds, fn)`, `tempRegexTimer` | One-shot timer.  `fn` must be a Lua function — Mudlet's code-as-string form is silently ignored (becomes a no-op callback); use `function() ... end`. |
| `tempTrigger(pattern, fn)`, `tempExactMatchTrigger`, `tempBeginOfLineTrigger` | Register triggers (Lua-pattern matching).  Function form only. |
| `killTrigger(id)`, `killTimer(id)`, `killAlias(id)` | Cancel by the id the registration returned. |
| `tempAlias(pattern, fn)` | Wraps `mud.alias`. |
| `registerNamedTimer(name, secs, fn, repeating)`, `stopNamedTimer`, `resumeNamedTimer`, `deleteNamedTimer`, `killNamedTimer`, `getNamedTimers` | Named-timer system with start/stop identity. |
| `registerNamedEventHandler`, `registerAnonymousEventHandler`, `raiseEvent`, `raiseGlobalEvent`, `killNamedEventHandler`, `killAnonymousEventHandler`, `deleteAllNamedEventHandlers`, `getNamedEventHandlers` | Working event bus. |
| `createStopWatch`, `startStopWatch`, `stopStopWatch`, `resetStopWatch`, `getStopWatchTime`, `deleteStopWatch` | Stopwatches. |
| `db:create(name, schema)` + table refs (`add`, `fetch`, `eq`, delete) | Sandboxed re-implementation of Mudlet's database layer.  Persists to a per-character file managed by the client (periodic flush + on close). |
| `string.split`, `string.trim`, `string.starts`, `string.ends`, `string.title`, `string.cut`, `string.findPattern`, `string.enclose` | Mudlet's string extensions, added to the `string` library. |
| `f"text {var}"` f-strings | Mudlet-style interpolation, **with limits** — see below. |
| `gmcp.*` | Lazy MSDP lookup via mapping table (see [gmcp table](#the-gmcp-table)). |
| `matches[]` | Populated before every trigger callback (see below). |
| `getProfileName`, `getMudletVersion`, `getMudletInfo`, `getConnectionInfo`, `getNetworkLatency`, `getMudletHomeDir`, `getPaths`, `exists` | Environment queries returning sensible CKMud values. |
| `downloadFile(path, url)` | Bridged to the host (sandboxed destination). |
| `require(name)` | Returns a benign auto-vivifying stub table (cross-module Mudlet `require` calls don't crash).  Does not load files. |
| `yajl.to_string(v)`, `yajl.to_value(s)` | JSON encode/decode (compact). |
| `getVariable(k)`, `setVariable(k, v)`, `mud.vars` | TinTin variable bridge — shared `@var` namespace. |
| `getConfig(name)`, `setConfig(name, v)`, `getEpoch()`, `unpack(t)` | In-memory config, epoch seconds, `table.unpack` alias. |
| Named captures `(?<name>)` + `multimatches[]` | Via the PCRE matcher / multi-line compat shim (see [PCRE triggers](#pcre--named-capture-triggers)). |

### f-string limits (read this if your script uses `f"..."`)

The shim implements `f"text {var}"` by scanning the **calling function's local variables** (plus globals) and supports dotted paths like `{player.name}`.  It does **not** see closure **upvalues**, and `{...}` cannot contain arbitrary expressions (`{a+b}`, `{t[1]}`, `{obj:method()}` won't evaluate).  When a name can't be resolved, the `{name}` text passes through literally — no error is raised.  If your interpolation references an upvalue or expression, switch to `string.format` or concatenation.

### Safe stubs (won't crash, won't really work)

These are callable and return neutral values so ported scripts keep running, but they have **no real effect** — CKMud has no editable screen-buffer model:

- Buffer/cursor manipulation: `selectString` (returns -1), `selectCaptureGroup`, `selectCurrentLine`, `selectSection`, `getFgColor`/`getBgColor` (return `255,255,255` / `0,0,0` so `g == 0` suppression checks don't fire falsely), `replace`, `replaceLine`, `deleteLine`, `moveCursor`, `moveCursorEnd`, `insertText`/`cinsertText`/`hinsertText`, `getSelection`, `getLines`, `getCurrentLine`, `setFgColor`/`setBgColor`/`fg`/`bg`/`resetFormat`. **For real screen colour, read MSDP instead.**
- Trigger toggling by name: `enableTrigger`/`disableTrigger` return `true` but don't change state; `deleteNamedTrigger`, `deleteNamedAlias` are no-ops.  Use `killTrigger(id)` with the registration id instead.
- `enableAlias`/`disableAlias`, `enableTimer`/`disableTimer` — same story; use ids.
- User windows / miniconsoles: `openUserWindow`, `echoUserWindow`, `clearWindow`, `closeUserWindow`, `createBuffer`, `appendBuffer`, `showWindow`, `hideWindow`, `setWindowWrap`, `setUserWindowOptions`.
- Misc: `feedTriggers`, `tempColorTrigger`, `expandAlias` (currently = `send`, does not re-run the alias matcher), `sendATCP`/`sendGMCP`/`sendMSDP`/`sendIRC`, `debugc`.

### Absent (calling these errors)

- **Geyser UI** (`Geyser.*`) — the entire widget framework.
- **Mudlet's `installPackage`** — use CKMud's `ckupdate` instead (see [ckupdate](#ckupdate--pull-script-updates-from-a-url)).
- **The `rex` PCRE library** (`rex.split`, etc.) — write a PCRE pattern in `mud.trigger`/`mud.triggerRegex` instead.
- **`gagLine`, `setTriggerStayOpen`** — not implemented.
- **True cross-line multi-line triggers** — single-line matching only (the `multimatches[]` compat shim covers the common anchor+capture idiom).

> **Previously listed as absent, now SUPPORTED:** `yajl` JSON, `getVariable`/`setVariable` + `mud.vars`, named-capture / PCRE triggers, `multimatches[]`, `getConfig`/`setConfig`, `getEpoch`, `unpack`.

### The `matches[]` table

Inside any trigger callback, the global `matches` table is populated with the same conventions Mudlet uses:

- `matches[1]` — the matched text (whole line for Lua patterns; the matched substring for PCRE)
- `matches[2]` — the first capture group
- `matches[3]` — the second capture group, … and so on
- `matches.NAME` — a **named** capture (PCRE patterns with `(?<NAME>...)` only)

The callback's positional args (`function(line, cap1, cap2, ...)`) still work too — use whichever style you prefer. `multimatches[n]` returns the same `matches` table for any `n` (Mudlet multi-line compat).

### The `gmcp` table

`gmcp.Some.Nested.Field` reads from CKMud's MSDP map via a hardcoded mapping table.  Standard fields are covered:

| `gmcp.*` path | Maps to MSDP key |
|---|---|
| `gmcp.Char.Vitals.hp` | `HEALTH` |
| `gmcp.Char.Vitals.maxhp` | `HEALTH_MAX` |
| `gmcp.Char.Vitals.mp` | `MANA` |
| `gmcp.Char.Vitals.maxmp` | `MANA_MAX` |
| `gmcp.Char.Status.level` | `LEVEL` |
| `gmcp.Char.Status.opponent` | `OPPONENT_NAME` |
| `gmcp.Char.Base.name` | `CHARACTER_NAME` |
| `gmcp.Room.Info.name` | `ROOM_NAME` |
| `gmcp.Room.Info.vnum` | `ROOM_VNUM` |
| `gmcp.Room.Info.area` | `AREA_NAME` |

Unmapped paths return `nil` and log a one-time warning to View Logs the first time the script reads them.  For non-standard fields, use `mud.msdp("KEY")` directly.

### Converting a Mudlet `.mpackage`

`.mpackage` files are renamed zip archives containing a profile XML that wraps each trigger/alias/timer's Lua body in `<script>` elements. CKMud has a built-in **Mudlet importer** (`MudletImporter`) that automates the conversion: it reads the XML and emits CKMud `.lua` files, translating Mudlet patterns to Lua patterns (or keeping them as PCRE when they use named groups / lookaround), extracting a literal keyword for each alias, and wiring triggers through `mud.trigger`/`mud.triggerRegex`. Triggers that genuinely can't run (Mudlet buffer-manipulation bodies, true multi-line) are emitted commented-out with a reason.

> **Caution — full-package imports can loop.** Some large packages (e.g. CK) run an init/refresh loop that requests `score`/`status`/`learn` and waits for a *parse* step to complete. If that parse relies on Mudlet buffer APIs CKMud only stubs, the loop never finishes and floods the MUD. The **runaway-send guard** stops the flood, but the stable approach is a *curated* set of `.lua` files (the pieces you actually use) served from your own repo via `ckupdate`, rather than importing the whole `.mpackage` and enabling everything.

Manual conversion (if you prefer): rename `.mpackage` → `.zip`, extract, and for each element copy its `<script>` body into a `.lua` file, wrapping trigger bodies in `mud.trigger("<pattern>", function(line, ...) <body> end)`. The shim handles `send`/`cecho`/`matches`/`gmcp`/etc.

---

## See also

- [USER_GUIDE.md](./USER_GUIDE.md) — Player-facing walkthrough
- [CLAUDE.md](../CLAUDE.md) — Architecture / build / engineering reference (developer-facing)
- In-app: Tools → **View Logs** for runtime trigger/alias diagnostic output
- In-app: type `#help` for the command quick-reference
