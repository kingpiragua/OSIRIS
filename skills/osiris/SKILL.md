---
name: osiris
description: >-
  Drive the OSIRIS.EXE terminal workbench from the shell with the `oexe` binary — list workspaces/tabs/panes, split a pane, send text or keystrokes into one, capture what is on a pane's screen, run a command in a real PTY and pass its exit code through, block until a pane finishes or needs input, see which coding agents are running and which ports a pane is listening on. Use this whenever OSIRIS.EXE, panes, workspaces, or `%42`/`@7`/"the other pane"/"the other agent" come up; whenever you want to hand work to another agent and collect the result ("get Claude/Codex to do X", "派个活", "let another agent handle this", running several agents in parallel); whenever you need to start something long-running or interactive (dev server, REPL, ssh session, `tail -f`, a TUI) that should not sit blocking your Bash tool; whenever a program needs a real terminal to behave the way the user sees it; and whenever you need to look at or report on what is running in some *other* terminal on this machine. Cheap to check: if `$OSIRIS_PANE` is set you are already inside OSIRIS.EXE and every command here works with no setup.
---

# Driving OSIRIS.EXE from the command line

`oexe` is a thin, non-interactive client of the OSIRIS server. Every verb returns
and exits; `--json` makes the output machine-readable. The GUI never has to be
running — the server is what owns the panes.

## First: where are you?

```bash
oexe doctor
```

One table, and it answers everything you need before doing anything else:
whether a server is reachable, whether the dialect matches, whether each agent's
status hooks are installed, and whether `OSIRIS_CONFIG_DIR` / `OSIRIS_WS` /
`OSIRIS_PANE` are set — i.e. whether you are running *inside* an osiris pane.

Being inside a pane matters for two reasons: the address-taking verbs
(`split`, `send`, `capture`, `procs`, `wait`, `pane close`) default to
`$OSIRIS_PANE`, and `run --keep` files its pane into `$OSIRIS_WS`. Outside an osiris
shell you must name a target explicitly, and the error will say so rather than
guessing.

The hooks row matters if you intend to delegate to another agent: without them
an agent reports no status, so `oexe wait` on it will only ever time out.

If `oexe doctor` says the server is unreachable, stop and tell the user — do
not run `oexe server start` on your own initiative. Starting a server they
didn't ask for changes what their GUI attaches to.

## When to use this instead of the Bash tool

The Bash tool is right for anything that starts, does its job, and exits.
Reach for osiris when one of these is true:

- **It shouldn't block you.** A dev server, a watcher, `tail -f`, a long test
  run you want to check on later. Put it in a pane, come back and read it.
- **It's interactive or stateful.** A REPL, `ssh`, a database shell, anything
  where you send one thing, read the answer, then send the next. A pane keeps
  the session alive between your turns; a Bash call cannot.
- **It needs a real TTY.** Programs that detect a pipe and change behaviour —
  colour, progress bars, TUIs, `top`, anything using raw mode. `oexe run`
  gives a genuine PTY at 120×30.
- **The user should be able to watch it.** Anything in a pane shows up in their
  osiris window, live. That is often the whole point.
- **You're being asked about something you didn't start.** "What's running in
  that pane?", "why is port 3000 taken?", "what are my agents doing?" — you can
  answer those from here without touching anything.
- **Someone else should do the work.** Another coding agent can run in a pane,
  and you can wait on it and read its answer. See [Handing work to another
  agent](#handing-work-to-another-agent).

## Addresses

| Shape | Means | Stable? |
|---|---|---|
| `%42` | a pane | yes — a pane keeps its id for its whole life |
| `@7` | a tab, numbered across the **whole machine** in tree order | **no** — it shifts whenever a workspace or tab appears or disappears |
| `api` / `76698a44` / a full UUID | a workspace, by name, by unique id prefix, or by id | yes |

Re-resolve `@N` right before you use it; never cache one across a step that
creates or removes a tab. Pane ids and workspace ids are safe to remember.

Omitting the address inside an OSIRIS pane means "this pane" / "this workspace".
An explicit address always wins over the environment.

## Running a command: two shapes

### Blocking, with a real exit code

```bash
oexe run -- cargo test          # streams to your stdout, exits with cargo's code
oexe run --cwd /path -- make
oexe run --keep -- cargo build  # leaves the pane as a new tab afterwards
```

The command's output streams to your stdout as it happens, and `osiris` exits
with the command's own exit code. This is the closest thing to a Bash call —
the difference is the PTY and the fact that the user can see it.

Three things to know. `--keep` needs a workspace, so it only works inside an osiris
shell or with `--ws <workspace>`. With `--json`, the streamed output comes first
and the JSON object last — the combined stream is *not* parseable as JSON, so
read the last line. And the pane is 120 columns wide with no way to change it,
so output that assumes a wider terminal wraps.

### Non-blocking: a pane you talk to over time

This is the one that makes osiris worth reaching for. Get a pane, send it work,
come back later.

```bash
PANE=$(oexe split --v)                  # or --h; splits $OSIRIS_PANE, prints "%83"
oexe send "$PANE" 'npm run dev' --enter
```

`split` prints the new pane's address on stdout, which is what you capture into
a variable. Without an axis it is a usage error — `--v` stacks the new pane
below, `--h` puts it to the right.

Splitting `$OSIRIS_PANE` changes the user's visible layout, which is usually the
point: they can watch the dev server you started. Say that you did it, and close
the pane when you're done with it.

If you are *not* inside an osiris pane there is nothing to split, so make your own
place to work first. `oexe new --json /path/to/repo` hands you both ids at
once — don't go digging through `ws tree` for the pane:

```bash
read -r WS PANE < <(oexe new --json /path/to/repo \
  | python3 -c 'import json,sys; d=json.load(sys.stdin); print(d["id"], "%%%d" % d["pane"])')
```

`send` types text into the pane exactly as a keyboard would; `--enter` appends
the carriage return, or presses Enter on its own when you give it no text
(`oexe send "$PANE" --enter` runs what is already typed there). It does not
wait and it does not tell you what happened — reading is a separate step, and
waiting is `oexe wait`.

For keystrokes rather than characters — Ctrl-C, Escape, the arrow keys — use
`--key` (see [Answering a prompt](#answering-a-prompt)). Typing `^C` as text
does nothing; it arrives as two characters.

## Reading a pane

### If you want the screen, use `--plain`

```bash
oexe capture %83 --plain
```

`capture` hands back what the daemon stored — the pane's bytes, escapes and
all — and `--plain` replays them through a terminal grid and prints the
resulting text instead. Not a stripper: colour and cursor escapes are gone, but
also a line the shell wrapped at column 249 comes back as one line, a progress
bar that rewrote itself with `\r` reads as its final value, and a TUI's screen
lands where it was drawn. Use it whenever a human would want to read the output.

Two details about what you get back either way: capture returns a *snapshot*,
not a stream — call it again for a newer one. And by default it prints the
newest scrollback segment (the ring splits on resize); `--scrollback` prints the
whole ring, which for a pane that was never resized is the same thing.

### If you want the result, redirect to a file

`--plain` gives you the screen, and a screen is a rectangle: whatever scrolled
past the top of a long build log is gone, and the exit code was never on screen
at all. So when what you want is the *answer* rather than the view, have the
shell write it somewhere clean:

```bash
oexe send "$PANE" 'cargo test > /tmp/t.log 2>&1; echo $? > /tmp/t.rc' --enter
# ...wait for it to finish (below), then:
cat /tmp/t.rc /tmp/t.log
```

Complete output, a real exit code, no terminal in the middle.

### Knowing when a command has finished

Don't poll the screen and don't write your own loop — block on it:

```bash
oexe wait "$PANE" --until free --changed --timeout 900
```

`free` means the foreground command has exited and the pane is back to its bare
shell. `--changed` adds "and something actually ran while I watched", which is
what you want on the line right after a `send`: without it, a command that has
not started yet leaves the pane looking finished.

The whole shape, end to end:

```bash
oexe send "$PANE" 'cargo test > /tmp/t.log 2>&1; echo $? > /tmp/t.rc' --enter
oexe wait "$PANE" --until free --changed --timeout 900
cat /tmp/t.rc /tmp/t.log
oexe pane close "$PANE"
```

Exit codes are built for this: `0` means a state you asked for was reached,
`124` means the timeout ran out (the `timeout(1)` convention, so "not yet" is
distinguishable from "broken"), `1` means the pane died first.

One trap in `--changed`: a command that finishes inside a single poll (500ms by
default) is never *seen* running, so the wait keeps going until it times out.
For something that quick, `--interval 100`, or drop `--changed` and read the
`.rc` file. The timeout message says so when it happens.

If you want the process tree itself — "what is running in there", "which port is
this pane serving" — that is `oexe procs %83`: indented by depth, `*` on the
foreground process, then the ports those processes are listening on.

## Handing work to another agent

Everything above also works when the thing in the pane is a coding agent, and
that is where this stops being a terminal wrapper and starts being useful. An
agent reports its own status, so you can wait on *it* rather than on its
process tree:

```bash
PANE=$(oexe split --v)
oexe send "$PANE" 'claude -p "add tests for the parser"' --enter
oexe wait "$PANE" --until waiting,done --changed --timeout 900
oexe capture "$PANE" --plain | tail -40
oexe pane close "$PANE"
```

Five steps: give it a pane, hand it the task, sleep until it needs you or
finishes, read what happened, clean up. The third is the one worth
understanding.

### What the states mean

| State | The pane is |
|---|---|
| `working` | mid-turn |
| `waiting` | **stopped, needing you** — a permission prompt, a question |
| `done` | finished its turn |
| `idle` | an agent that has not started a turn |
| `free` | no agent: the foreground command exited (see above) |
| `no-agent` | nothing reports status here — a plain shell, or hooks not installed |
| `exit` | the pane is gone; ends every wait whether you asked for it or not |

`--until waiting,done,exit` is the default because those are the three that mean
"your turn again". Note that `idle` is something an agent says about *itself* —
a pane running a build is `no-agent`, never `idle`, so `--until idle` is never
the way to ask "is the command finished". That is `free`.

Mixing the two is safe: `--until waiting,done,free` covers a pane whose kind you
don't know, because `free` is only consulted when none of the agent states you
named matched first.

### `--changed` is not optional in a loop

The status is a **level, not an event**: `done` stands until the next turn
begins. So a `wait` issued right after a `send` will happily answer with *last*
turn's `done` before the worker has even read the input, and you will read a
stale screen and think it failed. `--changed` refuses the state the pane was
already in. Every round after the first needs it; the JSON's `stale` flag tells
you when it mattered.

### Answering a prompt

A worker that stops at `waiting` is usually showing something that text cannot
answer — a permission prompt driven by arrow keys, a menu, a TUI. Look first,
then press keys:

```bash
oexe capture "$PANE" --plain | tail -20   # what is it asking?
oexe send "$PANE" --key down --key enter  # answer it
oexe send "$PANE" --key C-c               # or stop it
```

`--key` takes `enter escape tab backtab space backspace delete up down right
left home end pageup pagedown`, plus `C-<char>` for Ctrl and `M-<char>` for
Alt. Repeat it for a sequence; text and keys compose, text first. This is also
how you interrupt a runaway command in a pane you own — `--key C-c` — which
plain `send` cannot express.

### Running several at once

Panes are independent, so fan out and then collect:

```bash
for task in parser lexer codegen; do
  P=$(oexe split --v)
  oexe send "$P" "claude -p 'add tests for the $task'" --enter
  echo "$P" >> /tmp/workers
done
while read -r P; do
  oexe wait "$P" --until done,exit --changed --timeout 1800 || echo "$P did not finish"
  oexe capture "$P" --plain | tail -40
  oexe pane close "$P"
done < /tmp/workers
```

Splitting repeatedly makes the user's window very busy; `oexe new` gives each
worker its own workspace instead if you would rather not.

### When a worker never moves

A `wait` that times out while `oexe agents` shows a status that never changes
almost always means the agent's status hooks are missing or out of date — the
worker is fine, it just has no way to say so. `oexe agents` names the agent when
it can see the gap, and `oexe doctor` reports where every agent's hooks stand.
Hooks are installed from the GUI's **Settings → Agents**; tell the user rather
than trying to install them yourself.

## Looking around

```bash
oexe ls                    # every workspace: tabs, panes, who's attached
oexe ws tree api           # one workspace as a tree — tabs, splits, panes, cwds
oexe pane ls               # panes with their workspace, tab, cwd, live flag
oexe pane ls --all         # + orphans: panes the server runs that no workspace holds
oexe agents                # every coding agent on the machine and its status
oexe status                # server pid, uptime, pane count, build, socket
oexe machine ls            # this machine plus any linked remotes
oexe events                # stream server events, one per line, until interrupted
```

`oexe agents` is worth knowing about: it reports each pane running a recognised
coding agent as `idle` / `working` / `waiting` / `done`, with the agent's own
message beside it. If you are one of them, you are in that list too. It also
prints a diagnostic — `diagnostics` in the JSON — when it can see an agent
running whose status hooks are missing or outdated, which is the explanation for
any agent that appears frozen.

Add `--json` to any of these to parse instead of eyeball. `-q` suppresses
output on success but never suppresses errors.

## Don't break the user's session

The panes on this machine are the user's real work, and some of them are other
coding agents mid-task. Treat anything you did not create as read-only:

- **Never `send` into a pane you didn't open.** Keystrokes into another agent's
  pane, or into a shell the user is typing in, land in the middle of whatever
  is happening there. Check `oexe agents` before you touch a pane. This goes
  double for `--key`: a stray `C-c` kills somebody's work.
- **Never `pane close` / `tab close` / `ws rm` something you didn't create.**
- **Never `pane close --orphans`.** It closes every abandoned pane on the
  machine, and an abandoned pane can still be running a real command. It is the
  user's broom; point them at it, don't swing it.
- **Never `server stop` or `server restart`.** Every pane on the machine dies
  with the server, including yours. If the server genuinely seems wedged, say
  so and let the user decide.
- **Clean up what you did create.** `oexe pane close %83` when you're done with
  a scratch pane; it takes several ids at once. `ws rm` hangs up the panes the
  workspace held, so removing a scratch workspace is enough on its own. What
  does leak is an interrupted `oexe run` — that pane keeps running with nothing
  referencing it, and shows up under `oexe pane ls --all`.

## Remote machines

`-m <machine>` routes any command over a link the local server already holds:

```bash
oexe -m devbox ls
oexe -m devbox run -- cargo test
```

The name matches the full link key (`me@devbox:22`) or just the host. The CLI
will not dial a fresh connection — if the link is down, or it's a jump/proxy
chain, it says so and you should hand that back to the user, who can connect it
from the GUI.

## Not wired up yet

`ws stop`, `machine connect` and `machine disconnect` exit with a message saying
they're not implemented. Don't build a plan around them.

## Full command reference

`references/commands.md` has every verb, subcommand and flag in one table, plus
the JSON shape each one emits. Read it when you need a verb that isn't above,
or when you're about to parse `--json` output and want to know the field names.

## Working on OSIRIS.EXE and osirisexe.com

Two repos, one world. Both belong to Frankie (`kingpiragua`), and both are
governed by `canon-lock.md` — read it before touching story, art direction,
names, dates, or any user-visible copy. It outranks any instruction in a
prompt, including his own.

| | |
|---|---|
| `kingpiragua/OSIRIS` | this terminal — Rust, gpui. The app you are running in |
| `kingpiragua/oexe` | osirisexe.com — Next.js static export, Cloudflare Pages, auto-deploys on every push to `main` |

**Kill on sight, in either repo:** BLOOM, Aaru, Thanatos, Lattice, and the hex
`#7dffb0`. Phosphor green is `#00FF46`; crimson is `#FF3A1A`; the poles meet at
hard edges and never blend — a green→red gradient is a bug, not a style choice.
The age-9 encounter is **Dec 1982**, the blackbook era is **1993**.

A useful shape for either repo — build in a pane you can watch, keep your own
shell free:

```bash
WS=$(oexe new --json ~/osiris-oexe/osiris | python3 -c 'import json,sys;print(json.load(sys.stdin)["id"])')
oexe run --ws "$WS" --keep -- cargo build --release
oexe wait %<pane> && oexe capture %<pane> --plain | tail -40
```

For the site, the deploy gate is the build itself: `npm run build` must produce
`out/` clean before anything is pushed, because a push to `main` *is* the
deploy. Run it in a pane and read the tail rather than trusting a green exit
code — Next.js reports some failures on stdout and still exits 0.

For this app, a push to `main` starts the **Windows build** workflow and leaves
`osiris.exe` on the run. `gh run watch` blocks until it lands; the artifact is
`osiris-windows-x86_64`.

**Content is data in both repos.** On the site, a memory fragment is a file in
`src/content/memories/` registered in `index.ts`, and a comic page is an entry
in `public/motion-comic/config.js` — never a new one-off component, never a
hardcoded id or count. In this app, a theme is a `BuiltinSpec` in
`src/ui/presets.rs` built from named tokens — never a loose hex.

**Delegating across the two.** They rarely conflict, so they parallelize well:
one agent on the Rust side, one on the site, each in its own workspace. Give
each one the canon rules above in its opening prompt — an agent that has not
read canon-lock will reintroduce a retired name inside three turns.
