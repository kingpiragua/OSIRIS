# tty7 → OSIRIS.EXE

What this fork changed, why, and what to check the first time you build it.
Written so an upstream rebase stays boring: the diff is a rename plus a
palette, and every rule it follows comes from `canon-lock.md` v4.1.

## The shape of it

| | upstream | here |
|---|---|---|
| GUI binary | `tty7-app` (`tty7-app.exe`) | **`osiris`** (`osiris.exe`) |
| CLI on PATH | `tty7` | **`oexe`** |
| Session server | `tty7-server` | `osiris-server` |
| Update helper | `tty7-updater` | `osiris-updater` |
| Crates | `tty7`, `tty7-core`, `tty7-cli`, `tty7-server` | `osiris`, `osiris-core`, `osiris-cli`, `osiris-server` |
| Config dir | `~/.config/tty7` | `~/.config/osiris` · `%APPDATA%\osiris` |
| Pane env | `TTY7_PANE`, `TTY7_WS`, `TTY7_CONFIG_DIR`, … | `OSIRIS_PANE`, `OSIRIS_WS`, `OSIRIS_CONFIG_DIR`, … |
| `TERM_PROGRAM` | `tty7` | `osiris` |
| Keychain services | `tty7-ssh`, `tty7-ssh-key` | `osiris-ssh`, `osiris-ssh-key` |
| Windows AUMID | `com.github.tty7` | `com.diskdarian.osiris` |
| macOS bundle id | `com.github.tty7` | `com.diskdarian.osiris` |
| Inno `AppId` | tty7's GUID | a new GUID |
| Default theme | `light` | `osiris` |

**The CLI is `oexe`, not `osiris`.** The GUI executable owns the name
`osiris.exe`, and the installer stages both binaries into one directory — two
files cannot both be `osiris.exe`. `oexe` is short, it types quickly, and it is
the name the platform repo already uses.

Three renames were deliberately *not* made:

- `branch = "tty7"` in `Cargo.toml` / `Cargo.lock` — that is a real branch on
  the `gpui-component` and `zed` forks. Renaming it breaks the build.
- `osiris.app` — the macOS bundle directory name is a contract with
  `osiris-updater`, which looks for it by name when it replaces an install.
  The bundle's *display* name is `OSIRIS.EXE` (`CFBundleDisplayName`).
- The `OpenDiscord` action id, so anyone's existing keybinding still resolves.
  It opens `osirisexe.com` now; upstream's Discord is not this project's room.

Because identity changed everywhere, this build shares nothing with an
installed tty7: different config directory, different daemon socket, different
keychain entries, different uninstall entry. The two can sit on one machine
without touching each other, and a tty7 session will not migrate into it.

## Canon applied

`src/ui/presets.rs` carries the locked v3.0 tokens as named constants, and the
two OSIRIS themes are built from them — no loose hexes anywhere in the file:

```
--duat-black   #010103   --lake-midnight  #0B1226   --deep-indigo    #1B2A5E
--phosphor     #00FF46   --phosphor-dim   #0A8A2E   --crimson        #FF3A1A
--crimson-dim  #7A1A0C   --agi-gold       #FFC93C   --resurrection   #C9A227
--quantum-violet #6B3FA0 --bone           #D8D2C4   --cream          #D5CDB8
```

- **`osiris`** — the default. Phosphor on duat black, phosphor block cursor.
- **`eye_of_horus`** — the Network's apex interface: crimson chrome on the same
  black. The ANSI ramp is shared, so a program printing "green" still prints
  the Signal in either register. **The poles never blend** — the ANSI table has
  no slot that mixes them, no background is a green→red gradient, and slot 6
  (cyan) is phosphor-dim rather than a third pole.
- The nine upstream themes stay in the list, after these two.
- `theme_follow_system` names `osiris` on both sides: there is no light half of
  the archive to follow the system into.

Everything derived from a theme — hairlines, sidebar text, semantic inks, the
search wash — goes through upstream's contrast floors. Both OSIRIS themes were
checked against every floor the test suite enforces before they were added:
caret 15.3:1 and 5.8:1 on the background, hairlines at 1.60 and 1.55, sidebar
text 7.6:1 and 4.5:1, and the glyph on a search match at 4.76 and 3.00 against
a 3.0 floor.

**Voice.** English, Chinese and Japanese UI strings call the product
`OSIRIS.EXE` (or plain `OSIRIS` before a noun: "the OSIRIS server"). The
command palette's empty state is the shared error voice — `UNKNOWN COMMAND` /
`TYPE "help"`. No retired name — BLOOM, Aaru, Thanatos, Lattice — appears in
any string, asset or comment, and `#7dffb0` appears only where a comment names
it as banned. (`Cargo.lock` lists a crate called `module-lattice`: that is
russh's post-quantum key exchange, a third-party package name, not a word this
project chose.)

**Marks.** `assets/app-icon.svg` is a recovered CRT screen: duat black under
9-in-32 phosphor scanlines, the block cursor, and one integrity rule that runs
phosphor to 63% and stops dead against crimson — the Horus fractions, and the
only place in the mark where the two poles touch. `assets/app-icon-small.svg`
is a separate small-size cut (≤48 px) with the detail that cannot survive
downsampling removed; `favicon.ico` and `osiris.icns` are built from both, so
the taskbar icon is art-directed rather than averaged. The tray glyph is
alpha-only, as macOS template images require.

## Building osiris.exe

Push to `main` and the **Windows build** workflow
(`.github/workflows/windows.yml`) produces both artifacts on a Windows runner:

```
dist/osiris-<version>-windows-x86_64.zip          portable
dist/osiris-<version>-windows-x86_64-setup.exe    Inno installer
```

They land on the run as `osiris-windows-x86_64` — no tag, nothing published.
Tagging `v*` still runs the full release across Windows, macOS and Linux.

Locally on Windows (Rust + MSVC build tools):

```powershell
cargo build --release --locked --target x86_64-pc-windows-msvc
cargo build --release --locked --features updater --bin osiris-updater --target x86_64-pc-windows-msvc
./.github/scripts/bundle-windows.ps1 x86_64-pc-windows-msvc x86_64
```

The installed layout is `osiris.exe`, `oexe.exe`, `osiris-updater.exe`,
`completions\`, the bundled ConPTY pair, and the marker file the in-app updater
reads. `verify-windows-package.ps1` checks that list on every run.

## Verified here / left to CI

Built and tested in the conversion: `osiris-core` and `osiris-cli` compile
clean and their suites run. The rename broke exactly one thing, which is worth
knowing about if you write another one: `scrollback.rs` holds a fixed
**8-byte** file magic, and `TTY7SB\x01\x00` → `OSIRISSB\x01\x00` is ten. It is
`OSIRIS\x01\x00` now — six of name, then the version.

Not built here: the GUI crate, which needs the gpui fork and a desktop
toolchain. Every file touched in it was parse-checked, and its logic changes
are string literals, the two theme entries and one test. CI covers the rest.

Two follow-ups, neither blocking:

- **Geist Mono.** Canon's terminal voice is Geist Mono; the app still bundles
  Hack (`assets/fonts/hack`). Dropping Geist Mono's TTFs beside it and pointing
  `font_family` at it is the whole change — no CDN, the fonts are embedded.
- **CRT chrome.** Scanlines and burn-in live in the icon and the hero, not in
  the running window. A real phosphor overlay is a gpui shader change and
  should be gated: glitch is punctuation, not wallpaper.

## Rebasing on upstream

Add tty7 as a remote and take its changes; conflicts will almost all be the
rename, which `git` resolves by hunk. The files with genuine divergence, in the
order they will bite:

```
src/ui/presets.rs                     the tokens + two themes + one test
crates/osiris-core/src/core/config.rs default theme names
.github/scripts/*                     binary names in the packaging scripts
.github/workflows/windows.yml         new file, no upstream counterpart
build.rs  README.md  CONVERSION.md    identity
```
