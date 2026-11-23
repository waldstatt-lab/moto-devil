# Moto Devil

A stunt-bike game whose levels lie to you, built with [Bevy](https://bevy.org) 0.20
and [Rapier](https://rapier.rs). The project grew from a Bevy template and keeps its
documentation structure for shaping and recording work, agent instructions with a
library of Bevy skills, and [mise](https://mise.jdx.dev) tasks to build, test, and
run the game.

## What's inside

| Path | Purpose |
| --- | --- |
| `src/` | The game: headless simulation (with Rapier) plus presentation plugins |
| `web/` | The page for the browser build (`mise run web:build`) |
| `.github/workflows/` | CI checks and the GitHub Pages publication |
| `assets/fonts/` | The compiled-in UI font (DejaVu Sans Bold) and its license |
| `docs/idea.md` | Game direction and longer-term possibilities |
| `docs/pitches/` | Shaped feature proposals in German (Problem, Lösung, Sicherheit, Rabbit Holes, No-Gos) |
| `docs/architecture/` | Current structure and conventions, and [decision records](docs/architecture/decisions/README.md) |
| `.agents/skills/` | Bevy 0.20 skills for coding agents; versions in `skills-lock.json` |
| `shell.nix` | Native build and runtime libraries for NixOS (`nix-shell`) |
| `.mise/` | Pinned Rust toolchain and web tools, and `build`, `test`, `run`, `check`, `fmt`, `web:build`, `web:serve`, `agents:init-claude` tasks |
| `AGENTS.md`, `CONTRIBUTING.md` | Working brief for agents; workflow and validation for everyone |

## How to play

Type your name on the first screen (or pick it with ↑/↓) and press Enter. The level
map shows your way through the countries: finished levels have a tick, open ones
glow orange, and locked ones wait behind a padlock. Choose an open level with ←/→
and press Enter; the map starts at the first level you have not finished yet.
Finishing a level unlocks the next one, and you can ride any finished level again.
Ride to the flag; when the level lies
to you, you crash, respawn at the last checkpoint at once, and know one more trick.
Your gravestone marks the farthest place you crashed, next to everyone else's.

| Action | Keyboard | Controller |
| --- | --- | --- |
| Gas (in the direction the bike faces) | W or Up | Right trigger |
| Brake | S or Down | Left trigger |
| Tilt left / right (spin in the air for flips) | A / D or Left / Right | Left stick or D-pad |
| Jump / escape a cactus | Space | South (A / Cross) |
| Jet thrust (while fuel remains) | Hold Space | Hold South |
| Turn around | E | West (X / Square) |
| Restart the level | R | Select |
| Choose a level on the map | Left / Right | D-pad |
| Ride the chosen level | Enter | South |
| Back to the map after the finish | Enter | South |
| Back to the map while riding | Escape | Start |
| Back to the player selection from the map | Escape | East or Start |

Press Tab on the player selection to choose your own riding keys: pick an action
and a slot with the arrow keys, press Enter, then the new key. Backspace clears a
slot, the last row restores the keys above, and Escape goes back. Each action can
have two keys; the bindings are shared by everyone on this computer, and the HUD
shows them while riding. Escape, Enter, and the controller cannot be rebound.

Riding keys are physical positions, so WASD and E stay in place on any keyboard
layout. In the air the bike slowly rights itself when no tilt key is held, and on
the ground it never leans far enough to flip over. Escape on the player selection
quits; in the browser, close the tab instead.

The third level, **Fallen & Flugstunden**, introduces cactus, mud, diving ravens,
rising blocks, rolling and bouncing balls, trampolines, and short jet flights.
Traps can combine: one trampoline throws you into a cactus. Jump to escape, and
release the jet button to save fuel. Checkpoints reset traps and restore pickups.

After the training ground the game travels through five countries with three
levels each: Ägypten, Australien, USA, Paris (over the rooftops of the skyline),
and Schweiz. Each country has its own sky, landscape, and backdrop, and every level
keeps lying to you: blocks shoot out of the ground or fall out of the empty sky,
hatches give way, kangaroos hop at you, and the finish runs away. Egypt has
quicksand (do not stop, jump out), Switzerland has avalanches (outrun them, or
wait under a gallery roof).

Progress is saved in `moto-devil/progress.txt` in your data directory
(`~/.local/share` on Linux). Edit or delete that file to rename or remove players.
Key bindings are saved next to it in `keys.txt`; delete it to restore the defaults.

## Play in the browser

Every push to `main` publishes the game to GitHub Pages at
<https://waldstatt-lab.github.io/moto-devil/>. It runs in a current
desktop Firefox, Chrome, or Safari with WebGL2, played with the keyboard (or a
controller, if the browser reports it). Click the field if it does not react to
keys. Progress and keys are saved in the browser's local storage for that site, as
the items `moto-devil/progress` and `moto-devil/keys`; clearing the site's data
removes them. To try the browser build locally, run `mise run web:serve` and open
<http://127.0.0.1:8080>.

## Requirements

- [mise](https://mise.jdx.dev), recommended: `mise install` provides Rust 1.97.1
  with rustfmt, Clippy, and the `wasm32-unknown-unknown` target, plus
  `wasm-bindgen`, `wasm-opt`, and `miniserve` for the browser build, as pinned in
  [.mise/config.toml](.mise/config.toml).
  Without mise, install Rust 1.97.1 or newer (edition 2024) with rustup.
- Bevy 0.20.1 and bevy_rapier2d 0.37, downloaded automatically by Cargo.
  `Cargo.lock` pins dependencies.
- Clang and LLD on x86_64 Linux, used by [.cargo/config.toml](.cargo/config.toml)
  for faster linking. Remove that file to use the default linker instead.
- A working Vulkan (Linux, Windows) or Metal (macOS) graphics driver.

### Linux native libraries

Bevy needs ALSA, udev, X11 and Wayland libraries, and pkg-config. On Arch Linux:

```sh
sudo pacman -S --needed base-devel pkgconf alsa-lib systemd \
  libx11 libxcursor libxrandr libxi libxkbcommon wayland clang lld \
  vulkan-icd-loader
```

On Debian or Ubuntu:

```sh
sudo apt install build-essential pkg-config libasound2-dev libudev-dev \
  libx11-dev libxcursor-dev libxrandr-dev libxi-dev libxkbcommon-dev \
  libwayland-dev clang lld
```

On NixOS, adding these packages to your profile usually does not make them
visible to builds or the dynamic loader. If your system configuration does not
expose them to Cargo builds, use the project's [shell.nix](shell.nix); it provides
pkg-config, Clang, LLD, and the libraries, and puts the runtime libraries on
`LD_LIBRARY_PATH`. The Vulkan driver comes from the system
(`hardware.graphics.enable = true`):

```sh
nix-shell          # then: mise run run
nix-shell --run 'mise run check'
```

`nix-shell` is not entered automatically when you open the folder. To load it on
`cd`, install [direnv](https://direnv.net) with
[nix-direnv](https://github.com/nix-community/nix-direnv), add a `.envrc` containing
`use nix`, and run `direnv allow` once.

Install a Vulkan driver for your GPU on every distribution. If audio is routed through PipeWire, keep its ALSA plugin
(`pipewire-alsa`) installed.

See Bevy's [Linux dependency instructions](https://github.com/bevyengine/bevy/blob/latest/docs/linux_dependencies.md)
for other distributions and its [setup guide](https://bevy.org/learn/quick-start/getting-started/setup/).

## Development

Read [CONTRIBUTING.md](CONTRIBUTING.md) for the contribution workflow and validation,
the [architecture documentation](docs/architecture/README.md) for current structure
and conventions, and the [ADRs](docs/architecture/decisions/README.md) for recorded
architectural choices. AI coding agents should start with [AGENTS.md](AGENTS.md).

```sh
mise install     # once: install the pinned toolchain
mise run run     # build and start the game
mise run test    # headless tests
mise run build   # debug build; `mise run build --release` for release
mise run check   # format, Clippy, build, and tests before a PR
mise run web:build  # browser build into dist/web
mise run web:serve  # browser build, served on http://127.0.0.1:8080
mise tasks       # list every task
```

Each task passes extra arguments to Cargo, for example `mise run test jump`. The
tasks are plain scripts in [.mise/tasks/](.mise/tasks/), so the equivalent Cargo
commands also work:

```sh
cargo run --locked
cargo fmt --check
cargo clippy --locked --all-targets -- -D warnings
cargo build --locked
cargo test --locked
```

The first build downloads and compiles Bevy and can take several minutes.
Dependencies are optimized in development builds for usable runtime performance.

## Agent skills

`.agents/skills/` holds Bevy 0.20 skills from
[chrisgliddon/bevy-skills](https://github.com/chrisgliddon/bevy-skills): ECS, input,
cameras, UI, assets, audio, animation, physics, rendering, testing, WebAssembly,
and more, plus `similarity-rs` for finding duplicated Rust code. Start with the
`bevy` router skill. [skills-lock.json](skills-lock.json) records each skill's
source and content hash; update them together.

Codex and other agents that follow the `AGENTS.md` convention read these files
directly. Claude Code reads `CLAUDE.md` and `.claude/skills/` instead; link them to
the shared files once per checkout:

```sh
mise run agents:init-claude
```

This creates `CLAUDE.md -> AGENTS.md` and `.claude/skills -> ../.agents/skills`.
Rerunning it is safe, and it never replaces an existing file or a link that points
elsewhere.
