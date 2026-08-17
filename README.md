<div align="center">

<img src="assets/app-icon.svg" alt="OSIRIS.EXE" width="88" height="88" />

### OSIRIS.EXE

**The archive terminal: persistent sessions, remote work, agents.**

<sub>Pure Rust · GPU rendering on Zed's gpui · VT core from Alacritty</sub>

<br />

[![CI](https://github.com/kingpiragua/osiris/actions/workflows/ci.yml/badge.svg)](https://github.com/kingpiragua/osiris/actions/workflows/ci.yml)
[![Windows build](https://github.com/kingpiragua/osiris/actions/workflows/windows.yml/badge.svg)](https://github.com/kingpiragua/osiris/actions/workflows/windows.yml)
[![Version](https://img.shields.io/github/v/release/kingpiragua/osiris?label=version&color=00FF46)](https://github.com/kingpiragua/osiris/releases)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%C2%B7%20macOS%20%C2%B7%20Linux-blue)](https://github.com/kingpiragua/osiris/releases)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)

<sub>English · [简体中文](README.zh-CN.md)</sub>

<br />

<img src="assets/hero.webp" alt="OSIRIS.EXE: a phosphor pane listing workspaces, an agent turn completing, and the integrity rule along the bottom" width="900" />

<sub>Rendered from the shipped tokens — the chrome, not a photograph of it.</sub>

</div>

## What this is

A terminal workbench in OSIRIS.EXE's register. Same engine as its upstream —
GPU-rendered panes, a session server that outlives the window, native SSH, a
CLI built for coding agents — wearing the archive's own grammar instead of a
stock light theme:

- **Phosphor `#00FF46` is the Signal. Crimson `#FF3A1A` is the Network.**
  They meet at hard contact edges and never blend; a gradient between them
  would be deception in the story and a bug on screen.
- **Duat black `#010103` underneath**, bone and cream for the things that are
  only paperwork, gold and violet where the world uses them.
- **The block cursor is the only cursor**, and glitch is punctuation: nothing
  in the chrome animates just to prove it can.

Every color in `src/ui/presets.rs` is a locked v3.0 token spelled by name.
Nothing here invents a hex, and `#7dffb0` is banned on sight.

## Install

Windows, macOS and Linux builds on [**Releases**](https://github.com/kingpiragua/osiris/releases):

| | | |
|---|---|---|
| **Windows** | `…-setup.exe` · portable `….zip` | installs `osiris.exe` + the `oexe` CLI |
| **macOS** | `…-macos-arm64.dmg` · `…-x86_64.dmg` | drag into Applications |
| **Linux** | `…-x86_64.AppImage` | `chmod +x` and run — X11/Wayland libraries bundled |

Between releases, every push to `main` leaves a built `osiris.exe` on the
[Windows build](https://github.com/kingpiragua/osiris/actions/workflows/windows.yml)
run — open the newest green run and download `osiris-windows-x86_64`.

Building it yourself needs a Rust toolchain and, on Windows, the MSVC build
tools:

```sh
cargo build --release                       # osiris(.exe) and oexe(.exe)
cargo build --release --features updater --bin osiris-updater
```

## Three artifacts, one name

| | |
|---|---|
| `osiris` / `osiris.exe` | the GUI — the window, the panes, the archive chrome |
| `oexe` / `oexe.exe` | the CLI, on your PATH: drives panes with no GUI running |
| `osiris-server` | the headless session server, including on remote machines |

The CLI is `oexe` rather than `osiris` because the GUI owns that name now and
the two ship side by side in one directory. Everything the CLI does is in
[`docs/cli/reference.mdx`](docs/cli/reference.mdx) and
[`skills/osiris/SKILL.md`](skills/osiris/SKILL.md).

## What's inside

| | |
|---|---|
| **Editor-grade input** | ghost suggestions from history · explained tab completion · syntax highlighting · multi-line editing · click places the caret · <kbd>⌃ R</kbd> fuzzy history |
| **Window** | tabs & splits · <kbd>⌘ P</kbd> palette · <kbd>⌘ F</kbd> scrollback search · fourteen themes: five OSIRIS registers, then the nine upstream ones · IME |
| **Agent-aware** | per-pane detection (18 CLIs): status dot · notifications · branch + diff · resume after reboot · tray icon when input is needed |
| **Remote workspaces** | remote files, repos, changes, diffs, worktrees, tabs, and panes · reconnect from any client and continue where you left off |
| **CLI + Skills** | bundled `oexe` CLI · [agent skill](skills/osiris/SKILL.md) · pane/workspace control · real PTY commands · output, process, port, and agent status |
| **SSH** | native russh stack: profiles with keychain secrets · SFTP panel · port forwarding · jump hosts · one-time, unprivileged `osiris-server` install |

Full documentation lives in [**`docs/`**](docs/) —
[keyboard shortcuts](docs/reference/keyboard-shortcuts.mdx) ·
[config.json](docs/reference/configuration.mdx) ·
[CLI reference](docs/cli/reference.mdx).

## Themes

Five registers of one world, all built from the same locked tokens. None of
them blends the poles: a program printing green prints the Signal in every one.

| id | | |
|---|---|---|
| `osiris` | **OSIRIS.EXE** | the default — phosphor on duat black |
| `eye_of_horus` | **EYE OF HORUS.EXE** | the Network's side: crimson chrome, same black |
| `signal_bleed` | **Signal Bleed** | ink monochrome ground, phosphor fringe |
| `pale_horse` | **Pale Horse Cel** | night-blue cel ground, cream paint, crimson caret |
| `flyerhead` | **Flyerhead '93** | the xerox — ink on cream paper, crimson spot color |

`flyerhead` is the one light register, and it is light for a reason: a 1993 rave
flyer is ink on paper. Its ANSI ramp is swapped so a pane's own colors stay ink
instead of glowing.

The nine upstream themes stay in the list for when a screenshot has to look like
everyone else's terminal.

```jsonc
// ~/.config/osiris/config.json  (%APPDATA%\osiris\config.json on Windows)
{ "theme_preset": "osiris" }      // or eye_of_horus · signal_bleed · pale_horse · flyerhead
```

## Benchmarks

Inherited from upstream and unaffected by this fork — the renderer, VT parser
and session server are untouched. Same machine, same day, same 155×40 grid —
Apple M1 Pro, macOS 26.3.1, five-run averages (2026-07-04):

| | **OSIRIS.EXE** | Alacritty | Ghostty | Kitty |
|---|---:|---:|---:|---:|
| Plaintext I/O — 11 MB `cat` <sub>(lower = better)</sub> | **95 ms** | 239 ms | 179 ms | 185 ms |
| [DOOM-fire](https://github.com/const-void/DOOM-fire-zig) frame rate <sub>(higher = better)</sub> | **888 fps** | 485 fps | 552 fps | 617 fps |
| Cold-launch memory | 116 MB¹ | 105 MB | 128 MB | 130 MB |

<sub>¹ GUI 105 MB + the persistent server 11 MB.</sub>

Methodology and one-command reproduction: [`scripts/bench/`](scripts/bench/README.md).

## Credit

OSIRIS.EXE is a fork of [**tty7**](https://github.com/l0ng-ai/tty7) by the tty7
contributors, used under the Apache-2.0 license both projects carry. The
engineering is theirs; the archive is ours. Upstream fixes rebase cleanly
because this fork is a rename plus a palette, not a rewrite — see
[`CONVERSION.md`](CONVERSION.md) for exactly what changed.

---

<div align="center">
<sub>

Built on [gpui](https://github.com/zed-industries/zed) and [`alacritty_terminal`](https://github.com/zed-industries/alacritty) · forked from [tty7](https://github.com/l0ng-ai/tty7) · [Apache-2.0](LICENSE) · [Changelog](CHANGELOG.md)

</sub>
</div>
