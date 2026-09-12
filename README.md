# Ghostty Session Restore on Linux

Make [Ghostty](https://ghostty.org) reopen with all tabs, panes, and working
directories intact after a crash, a Ghostty update, or a full reboot —
on Linux, where Ghostty's native `window-save-state` is not yet
implemented (tracked in
[ghostty-org/ghostty#1847](https://github.com/ghostty-org/ghostty/issues/1847)).

The idea: let [zellij](https://zellij.dev) own the session and have
Ghostty attach to it on every launch.

## Minimum requirements

- Linux with a graphical session
- **Ghostty** with the `command = …` config option (any modern release)
- **zellij ≥ 0.40** with session serialization support (tested with 0.44.3)
- `zellij` should be installed on a **system** `PATH`, such as
  `/usr/local/bin`. A desktop-launched Ghostty inherits its `PATH` from
  the systemd user manager, and that one often has no `~/.local/bin` and
  no `~/.cargo/bin`. A zellij that lives only there can work from a
  terminal and still fail from the app grid with `sh: zellij: not found`.
  Check what yours has with
  `systemctl --user show-environment | grep ^PATH`.
  - If Ghostty is installed as a snap, it must be **classic** confinement,
    not strict — check with `snap info ghostty | grep confinement`

## Install zellij

Pick whichever you prefer:

```sh
# Option A: official static binary
curl -fsSL -o /tmp/z.tar.gz \
  https://github.com/zellij-org/zellij/releases/latest/download/zellij-x86_64-unknown-linux-musl.tar.gz
tar -xzf /tmp/z.tar.gz -C /tmp
sudo install -m 755 /tmp/zellij /usr/local/bin/zellij

# Option B: package manager
sudo pacman -S zellij        # Arch
# brew install zellij        # Linuxbrew
```

`cargo install --locked zellij` works too, but it lands in
`~/.cargo/bin`. Link it into place afterwards:
`sudo ln -s ~/.cargo/bin/zellij /usr/local/bin/zellij`.

Confirm `zellij --version` works and that `which zellij` prints a system
path.

## Configure

### `~/.config/ghostty/config.ghostty`

```ini
command = zellij attach --create --force-run-commands main
```

- `--create` — attach to a session named `main`, create it if missing.
- `--force-run-commands` — on resurrection, re-run each pane's last
  foreground command (e.g. `nvim file.py`). Plain shells are unaffected.

### `~/.config/zellij/config.kdl`

```kdl
session_serialization true
serialize_pane_viewport true
scrollback_lines_to_serialize 10000

// Alt+click a file path to open it in its desktop default application.
scrollback_editor "setsid -f xdg-open"

// Free Ctrl+O and Ctrl+B so those keys reach the running program.
keybinds {
    shared_except "session" "locked" {
        unbind "Ctrl o"
        bind "Ctrl y" { SwitchToMode "Session"; }
    }
    session {
        unbind "Ctrl o"
        bind "Ctrl y" { SwitchToMode "Normal"; }
    }
    shared_except "tmux" "locked" {
        unbind "Ctrl b"
    }
}
```

The three serialization options are off by default in zellij 0.44.3 and
must be set explicitly. Serialized state lives in
`~/.cache/zellij/<version>/<session>/`.

`scrollback_editor` is the command zellij runs when you Alt+click a file
path in a pane. Left unset it falls back to `$EDITOR`, then `$VISUAL`,
then plain `vi`, so the file lands in a terminal editor. Pointing it at
`setsid -f xdg-open` hands the file to whatever desktop application owns
that file type instead. Clicking a *folder* is unaffected and still
opens zellij's built-in file browser.
See [SETUP.md](SETUP.md#altclick-on-a-path-opens-the-file-in-a-desktop-app)
for how that works and what it costs.

The `keybinds` block exists because zellij intercepts its mode keys
before the program inside the pane sees them, and its defaults claim
`Ctrl+ b c f g h n o p q s t`. Two of those shadow Claude Code:
`Ctrl+O` (Session mode) is its expand-tool-output / transcript toggle,
and `Ctrl+B` (tmux compatibility mode) backgrounds a running process.
Session mode is *moved* to `Ctrl+Y` so detach and the session manager
stay reachable; tmux mode is *unbound* outright, since everything it
offers is already on another mode key.
See [SETUP.md](SETUP.md#why-the-keybinds-block-zellij-eats-the-apps-ctrl-keys)
for the general recipe.

## Activate

Close every Ghostty window and open a new one. The `command =` option
only applies to newly spawned terminals; `ghostty +reload-config` will
not re-shell windows that are already open.

## How it behaves

| Scenario | Result |
|---|---|
| Ghostty crash | Next window reattaches instantly; processes still running. |
| Ghostty self-update / restart | Same as crash. |
| Reboot | Layout, tabs, panes, cwds, and last foreground commands are restored. In-flight background processes are gone — impossible to preserve across a kernel restart. |

## Trade-off

Tabs and splits now belong to zellij, not Ghostty. zellij is modal —
the bottom status bar shows the current mode and available keys.

| Action | Keys |
|---|---|
| New tab | `Ctrl+t` then `n` |
| Switch tab | `Ctrl+t` then `h` / `l` |
| Reorder tab left / right | `Alt+i` / `Alt+o` |
| New pane (right / down) | `Ctrl+p` then `r` / `d` |
| Move focus | `Ctrl+p` then arrow / `hjkl` |
| Detach (keep session alive) | `Ctrl+y` then `d` |
| Session manager | `Ctrl+y` then `w` |
| Open a file or folder path in the pane | `Alt`+click |
| Quit zellij | `Ctrl+q` |

Reordering tabs is an `Alt` binding, not a Tab-mode action — Tab mode
only navigates and manages tabs, so there is no `Ctrl+t` equivalent.

A tab showing `(SYNC)` is in sync mode — every keystroke goes to *all*
panes in that tab. It's `Ctrl+t` then `s`, easy to hit by accident next
to `r` and `x`; press it again to clear.

Session mode is `Ctrl+y` because of the rebind above; on a stock zellij
it is `Ctrl+o`. Stock zellij also answers `Ctrl+b` (tmux compatibility
mode); this config unbinds it.

## Uninstall

```sh
# 1. In ~/.config/ghostty/config.ghostty, comment out the `command =` line.
# 2. Close all Ghostty windows.

zellij kill-all-sessions
rm -rf ~/.cache/zellij ~/.config/zellij ~/.local/share/zellij
sudo rm -f /usr/local/bin/zellij
```

## Local docs

This repo also includes a snapshot of Ghostty's configuration docs for
offline reference:

- `01-overview.md` — docs map
- `02-config-syntax.md` — config file location and `key = value` syntax
- `03-config-reference.md` — full option reference
- `04-keybindings.md` — keybind syntax and actions
- `05-themes.md` — themes
- `config.example` — annotated starter config
- `SETUP.md` — full setup log with decisions and rationale
